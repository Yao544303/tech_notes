# Spark一些知识点

## reduceByKey 是否会触发shuffle

根据

```
if (self.partitioner == Some(partitioner)) 
```

来决定是否生成ShuffledRDD。其中self.partitioner是指A这个RDD的partitioner，它指明了A这个RDD中的每个key在哪个partition中。而等号右边的partitioner，指明了B这个RDD的每个key在哪个partition中。当二者==时，就会用self.mapPartitions生成MapPartitionsRDD， 这和map这种transformation生成的RDD是一样的，此时reduceByKey不会引发shuffle。 [REF](https://www.cnblogs.com/devos/p/4795338.html)

## Spark WebUI 资源显示问题

在Spark Web UI 中 Executor 项下，可以看到一个Summary

```
	RDD Blocks	Storage Memory	Disk Used	Cores	Active Tasks	Failed Tasks	Complete Tasks	Total Tasks	Task Time (GC Time)	Input	Shuffle Read	Shuffle Write	Blacklisted
Active(41)	0	0.0 B / 127.1 GB	0.0 B	40	0	0	831	831	37.7 m (1.1 m)	985.5 MB	4.0 GB	1412.0 MB	0
Dead(0)	0	0.0 B / 0.0 B	0.0 B	0	0	0	0	0	0 ms (0 ms)	0.0 B	0.0 B	0.0 B	0
Total(41)	0	0.0 B / 127.1 GB	0.0 B	40	0	0	831	831	37.7 m (1.1 m)	985.5 MB	4.0 GB	1412.0 MB	0
```

发现其中分为 Active Dead 和 Total 三项

那为什么会有Dead 项呢？ Dead 就是在Stage中，执行完一部分任务，然后集群认为已经ok了，不需要这些Executor了，可以kill 了 ，所以就是Dead Active 与最后执行的Stage 中的Executor 数量一致，（cluster 模式 +1 为driver)

以上是在动态分配资源的模式可用，在固定资源的情况下，从头到尾都是一致的。

## 广播变量的读取

广播变量的读取比较复杂，首先读取端会尝试从本地BlockManager直接读取为切分的完整数据；如果不存在会尝试从本地BlockManager读取切分的数据块；如果都不存在，则从远端的driver或executor拉取，拉取每个数据块时，都会随机选择一个持有该数据块的executor或driver进行拉取，这样可以减少各个节点的网络IO压力。远端拉取来数据块会拷贝一份存储在本地BlockManager，以便其他executor拉取数据用。如果广播变量是读取数据块，会将数据块拼回完整数据对象，然后会将完成的数据对象拷贝一份存储在本地BlockManager，以便executor上执行的tasks快速读取广播变量。 由此可以看出广播变量会在每个节点存储两份：

* 一份是未切分的完整数据对象，用于executor或driver上执行的tasks快速读取
* 一份是切分后的数据，用于其他executor拉取对应的数据块。 spark的广播变量的写入比较简单，写入本地BlockManager两份数据即可。读取比较复杂，这里也真正的体现了p2p的BitTorrent协议的实现。 [spark源码分析— spark广播变量](https://blog.csdn.net/Shie\_3/article/details/80214035)

## Spark划分job 和stage

* 所谓一个Job,就是一个rdd 的action 触发的动作，可以简单理解为，当你需要执行一个rdd的action 的时候，会生成一个Job
* stage 是一个job的组成单位，一个job会被切分成一个或一个以上的stage，然后各个stage会按照顺序依次执行。
* task即 stage 下的一个任务执行单元，一般来说，一个 rdd 有多少个 partition，就会有多少个 task，因为每一个 task 只是处理一个 partition 上的数据。
* repartition 触发shuffle，处理分布不均的情况
* coalesce 不会触发shuffle,只能多变少，不能处理分布不均的情况

## yarn-cluster 和 yarn 的区别

yarn-cluster 的driver 为rm yarn 的driver 为local 也就是说 yarn 为 yarn-client

总结一下Spark中各个角色的JVM參数设置：

1. Driver的JVM參数： -Xmx。-Xms，假设是yarn-client模式，则默认读取spark-env文件里的SPARK\_DRIVER\_MEMORY值，-Xmx，-Xms值一样大小；假设是yarn-cluster模式。则读取的是spark-default.conf文件里的spark.driver.extraJavaOptions相应的JVM參数值。

PermSize，假设是yarn-client模式，则是默认读取spark-class文件里的JAVA\_OPTS="-XX:MaxPermSize=256m $OUR\_JAVA\_OPTS"值；假设是yarn-cluster模式。读取的是spark-default.conf文件里的spark.driver.extraJavaOptions相应的JVM參数值。 GC方式，假设是yarn-client模式，默认读取的是spark-class文件里的JAVA\_OPTS。假设是yarn-cluster模式，则读取的是spark-default.conf文件里的spark.driver.extraJavaOptions相应的參数值。 以上值最后均可被spark-submit工具中的--driver-java-options參数覆盖。 2. Executor的JVM參数： -Xmx，-Xms。假设是yarn-client模式，则默认读取spark-env文件里的SPARK\_EXECUTOR\_MEMORY值，-Xmx。-Xms值一样大小。假设是yarn-cluster模式，则读取的是spark-default.conf文件里的spark.executor.extraJavaOptions相应的JVM參数值。 PermSize。两种模式都是读取的是spark-default.conf文件里的spark.executor.extraJavaOptions相应的JVM參数值。 GC方式。两种模式都是读取的是spark-default.conf文件里的spark.executor.extraJavaOptions相应的JVM參数值。

1. Executor数目及所占CPU个数 假设是yarn-client模式。Executor数目由spark-env中的SPARK\_EXECUTOR\_INSTANCES指定，每一个实例的数目由SPARK\_EXECUTOR\_CORES指定；假设是yarn-cluster模式。Executor的数目由spark-submit工具的--num-executors參数指定，默认是2个实例，而每一个Executor使用的CPU数目由--executor-cores指定，默觉得1核。

[Spark On Yarn遇到的几个问题](https://www.cnblogs.com/mthoutai/p/6962242.html)

## Spark要很好的支持SQL，要完成解析(parser)、优化(optimizer)、执行(execution)三大过程。

处理顺序大致如下：

1. SQlParser生成LogicPlan Tree；
2. Analyzer和Optimizer将各种Rule作用于LogicalPlan Tree；
3. 最终优化生成的LogicalPlan生成SparkRDD；
4. 最后将生成的RDD转换序列交由Spark执行；

## SparkSQL的快

SparkSQL 快根本不是刚才说的那一坨东西哪儿比Hive On MR快了，而是Spark引擎本身快了。 事实上，不管是SparkSQL，Impala还是Presto等等，这些标榜第二代的SQL On Hadoop引擎，都至少做了三个改进，消除了冗余的HDFS读写，冗余的MapReduce阶段，节省了JVM启动时间。 在MapReduce模型下，需要Shuffle的操作，就必须接入一个完整的MapReduce操作，而接入一个MR操作，就必须将前阶段的MR结果写入HDFS，并且在Map阶段重新读出来，这才是万恶之源。 一次HDFS中间数据写入，其实会因为Replication的常数扩张为三倍写入，而磁盘读写是非常耗时的。这才是Spark速度的主要来源。 另一个加速，来自于JVM重用。考虑一个上万Task的Hive任务，如果用MapReduce执行，每个Task都会启动一次JVM，而每次JVM启动时间可能就是几秒到十几秒，而一个短Task的计算本身可能也就是几秒到十几秒，当MR的Hive任务启动完成，Spark的任务已经计算结束了。对于短Task多的情形下，这是很大的节省。

## Spark 2.0 对TPC-DS的支持度好了很多 可以运行99个SQL

在Catalyst中为不同的SQL功能增加相应的变换器，将其变成有相同语义的已经实现的SQL执行计划。这个实现方式总体上不错，但同时也有很多的局限性，譬如比较难以处理各种corner case，对子查询的支持比较差，只能运行简单场景下的子查询，此外一些SQL可能会触发多种变换器，从而带来一些不是期望中的结果。 在我们的测试中也发现，对于非TPC-DS中的SQL业务，譬如涉及到非等值的关联/子查询等，Spark SQL仍然不支持。因此，对于复杂的生产环境，尤其是基于复杂的主题模型设计出来的企业业务

## Spark 相比MapReduce 优化的地方

1. DAG编程模型。将数据加工链路整理成一个DAG(有向无环图)，这其实在理论上给hadoop数据加工过程一个很好的总结，对于数据加工链路比较长的job，spark会将其切分成多个stage对应了Hadoop的多个Map和Reduce逻辑，一个stage包含了多个并行执行的tasks。stage之间通过shffle传递数据，通过依赖相互串联起来。这样的话整个job过程只需要在Hdfs读写一次，不需要像mapreduce那样在hdfs中间写入多次，大大提高了速度。
2. 申请资源方式不同。spark上Excutor对应一个Jvm资源，多个task对应Jvm上的多个线程，Excutor可以被多个task所复用，申请资源次数比较少。Mapreduce每一个task任务会对应一个进程，且相互之间不能复用，申请资源次数比较多。
3. RDD缓存机制。spark对于中间运算的结果可以缓存在内存，下次再取数据时就不必重新计算，这非常适用于一些迭代式的任务的运行，可以大大提高速度。
4. 容错机制。spark在DAG的基础上建立了每个RDD的血统依赖和检查点机制。当数据加工过程中的某个task中间执行失败后，可以根据其依赖关系以最小代价重启task，而不必像Mapreduce那样从头开始运行task。

## Spark Vs Hadoop MR

spark框架之所以比hadoop mapreduce这套框架优秀，很重要的一点就是spark框架给出了数据加工处理流程一些清晰而极具抽象的概念。根据这些概念，我们可以将数据加工流程很容易的划分成一个阶段一个阶段的（对应与spark的stage）,这有点类似于数据加工的标准化，相比较而言mapreduce还是比较粗糙和混沌的，这在一些数据加工链比较长的作业体现的更加明显，从这点来说，spark对于大数据而言意义还是比较大的。 spark提出的一个新的概念是RDD（Resilient Distributed Datasets: A Fault-Tolerant Abstraction for In-Memory Cluster Computing），中文意思就是弹性数据集，提出来源是Berkeley实验室，从这点来看带有浓郁的学术色彩。RDD是spark数据计算的最小单元，我们做各种数据操作，比如过滤，加减等对应的都是从一个RDD到另一个RDD，前边提到的stage的构成就是由一系列的RDD构成，只不过这些RDD一般都满足特定的规律。 RDD是spark中的一个类，主要包括5个重要属性，这5个属性基本涵盖了分布式运算中常用的操作，分别如下所示。

* 分区列表，这是做分布式计算首先考虑的问题，考虑到分布式运算其实是将数据平均打散到各个节点或者各个jvm里边中，体现到数据上就是每个分区，合理的控制分区的数据集是分布式运算一个比较重要的参考指标，分区列表中的每个分区对应了spark中的一个task.
* 分区器，这个和分区列表相对应的，主要指定了分区的具体策略，上边提到了将数据平均打散到各个分区内，这里就是负责数据打散的策略。
* 依赖列表，spark加工的数据链路一般比较长，每一次的数据加工都会生成一个RDD，利用依赖列表可以有效的将RDD串联起来。
* 分区计算函数，获得分区数据后，会对分区数据进行相关操作，这也是真正进行数据处理的一步。每个RDD的分区计算逻辑是不同的，如果当前RDD没有父RDD例如HadoopRDD就会直接取出每条数据并以迭代器的形式返回最终的数据；如果当前RDD含有父RDD，则会首先获取父RDD的迭代器，这个过程需要传入当前的分区作为形参并可以递归的执行直到当前RDD没有父RDD，需要注意的是这个递归过程的执行是个参数反向传递的过程，从当前stage所依赖的RDD一直传导到上一个stage的结束的RDD。
* 位置信息，记录每个数据分区所在的数据块位置，例如在Hdfs中记录了数据块所在的节点IP，这个属性主要在task分发到相应的excutor上中使用。

## RDD模型适合的是粗粒度的全局数据并行计算，不适合细粒度的、需要异步更新的计算。
