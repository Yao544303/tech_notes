# Spark On Yarn 内存分配的机制

## 综述

在提交任务时，配置的executor-memory 的参数，设置为6g，结果实际运行中，占据的资源算下来，每个executor 使用的却接近7个g，被管理集群的同事找上门，逃。 那么，为何会配置的参数没有生效呢？

## executor-memory 工作生效机制

Spark提交任务首先由客户端（Client）生成一个Spark Driver，Spark Driver在YARN的NodeManager中生成一个ApplicationMaster，ApplicationMaster向YARN Resourcemanager申请资源（这些资源就是NodeManager封装的Container，其中包含内存和cpu），接着Spark Driver调度并运行任务。 在Cluster模式下，提交完任务，Client就可以离开了。而在Client模式下，Client不能离开。

Spark的Excutor的Container内存有两大部分组成：堆外内存和Excutor内存。其中

* 堆外内存，由spark.yarn.executor.memoryOverhead参数设置。  主要用于JVM自身的开销。默认：MAX(executorMemory\*0.10,384MB) 这里384 的含义为，JVM自身开销，最小的需求是384MB
* Excutor内存，由spark.executor.memory参数设置，分为两部分。
  * Execution:shuffle、排序、聚合等用于计算的内存
  * Storage：用于集群中缓冲和传播内部数据的内存（cache、广播变量）

## 实际分配Executor 内存的计算公式

关于Executor 计算的相关公式，见源码org.apache.spark.deploy.yarn.Clent，org.apache.spark.deploy.yarn.ClentArguments 主要部分如下

```
var executorMemory = 1024 // 默认值，1024MB
val MEMORY_OVERHEAD_FACTOR = 0.10  // OverHead 比例参数，默认0.1
val MEMORY_OVERHEAD_MIN = 384 

val executorMemoryOverhead = sparkConf.getInt("spark.yarn.executor.memoryOverhead",
math.max((MEMORY_OVERHEAD_FACTOR * executorMemory).toInt, MEMORY_OVERHEAD_MIN))
// 假设有设置参数，即获取参数，否则使用executorMemoryOverhead 的默认值

val executorMem = args.executorMemory + executorMemoryOverhead
// 最终分配的executor 内存为 两部分的和
```

好，现在回到我们的问题中，我们的提交命令为：

```
spark-submit --master yarn-cluster --name test --driver-memory 6g --executor-memory 6g
```

设置的executor-memory 大小为6g，executorMemoryOverhead为默认值，即max(6g\*0.1,384MB)=612MB 那总得大小应该为6144MB+612MB=6756MB 然而实际的开销为 7168 MB为什么？ 这就涉及到了规整化因子。

### 规整化因子介绍

&#x20; 为了易于管理资源和调度资源，Hadoop YARN内置了资源规整化算法，它规定了最小可申请资源量、最大可申请资源量和资源规整化因子，如果应用程序申请的资源量小于最小可申请资源量，则YARN会将其大小改为最小可申请量，也就是说，应用程序获得资源不会小于自己申请的资源，但也不一定相等；如果应用程序申请的资源量大于最大可申请资源量，则会抛出异常，无法申请成功；规整化因子是用来规整化应用程序资源的，应用程序申请的资源如果不是该因子的整数倍，则将被修改为最小的整数倍对应的值，公式为ceil(a/b)\*b，其中a是应用程序申请的资源，b为规整化因子。

比如，在yarn-site.xml中设置，相关参数如下：

```
yarn.scheduler.minimum-allocation-mb：最小可申请内存量，默认是1024
yarn.scheduler.minimum-allocation-vcores：最小可申请CPU数，默认是1
yarn.scheduler.maximum-allocation-mb：最大可申请内存量，默认是8096
yarn.scheduler.maximum-allocation-vcores：最大可申请CPU数，默认是4
```

对于规整化因子，不同调度器不同，具体如下： FIFO和Capacity Scheduler，规整化因子等于最小可申请资源量，不可单独配置。 Fair Scheduler：规整化因子通过参数yarn.scheduler.increment-allocation-mb和yarn.scheduler.increment-allocation-vcores设置，默认是1024和1。

通过以上介绍可知，应用程序申请到资源量可能大于资源申请的资源量，比如YARN的最小可申请资源内存量为1024，规整因子是1024，如果一个应用程序申请1500内存，则会得到2048内存，如果规整因子是512，则得到1536内存。 具体到我们的集群而言，使用的是默认值1024MB，因而最终分配的值为 ceil(6756/1024)\*1024 = 7168

## Client 和 Cluster 内存分配的差异？

在使用Clietn 和 Cluster 两种方式提交时，资源开销占用也是不同的。

不管CLient或CLuster模式下，ApplicationMaster都会占用一个Container来运行；而Client模式下的Container默认有1G内存，1个cpu核，Cluster模式下则使用driver-memory和driver-cpu来指定；

cluster 提交命令

```
spark-submit --master yarn-cluster --name testClient --driver-memory 6g --executor-memory 6g  --num-executors 10
```

一共10个executor，加上一个am，共11个，每个都分配了7g，即7_1024_11=78848

client 提交命令

```
spark-submit --master yarn-client --name testClient --driver-memory 6g --executor-memory 6g  --num-executors 10
```

同样是10个executor，每个分配7g，但是am只有1g，因而占用 7_1024_10+1\*1024=72704

Tips：在Client 模式下，如何设置driver-memory

涉及参数

```
yarn.scheduler.maximum-allocation-mb
这个参数表示每个container能够申请到的最大内存，一般是集群统一配置。Spark中的executor进程是跑在container中，所以container的最大内存会直接影响到executor的最大可用内存。当你设置一个比较大的内存时，日志中会报错，同时会打印这个参数的值。如下图 ，6144MB，即6G
spark.yarn.exeuctor.memoryOverhead
executor执行的时候，用的内存可能会超过executor-memoy，所以会为executor额外预留一部分内存。spark.yarn.executor.memoryOverhead代表了这部分内存。这个参数如果没有设置，会有一个自动计算公式(位于ClientArguments.scala中)
```

其中，MEMORY\_OVERHEAD\_FACTOR默认为0.1，executorMemory为设置的executor-memory, MEMORY\_OVERHEAD\_MIN默认为384m。参数MEMORY\_OVERHEAD\_FACTOR和MEMORY\_OVERHEAD\_MIN一般不能直接修改，是Spark代码中直接写死的。

## REF

* [Spark on YARN占用资源分析 - Spark 内存模型](https://blog.csdn.net/wjl7813/article/details/79971791)
* [Spark On YARN内存和CPU分配](https://blog.csdn.net/fansy1990/article/details/54314249)
