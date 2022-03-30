# Spark多任务参数调优经历

## 综述

在开发完Spark作业之后，就该为作业配置合适的资源了。Spark的资源参数，基本都可以在spark-submit命令中作为参数设置。很多Spark初学者，通常不知道该设置哪些必要的参数，以及如何设置这些参数，最后就只能胡乱设置，甚至压根儿不设置。资源参数设置的不合理，可能会导致没有充分利用集群资源，作业运行会极其缓慢；或者设置的资源过大，队列没有足够的资源来提供，进而导致各种异常。总之，无论是哪种情况，都会导致Spark作业的运行效率低下，甚至根本无法运行。 本文记录了一次，由于Spark任务阻塞引起的资源调优的过程，叙述了一些在Spark调优过程中会使用的方法，以供大家参考。

## 背景说明

我们使用第三方的数据和计算资源，这里需要说明，因为某些不可抗的因素，数据和计算资源都不在我们的控制范围内，只能通过特定的方式，将程序更新至第三方机器，通过crontab每天定时起调度。 原始数据按照不同来源，分小时存放。每个来源，每个小时的数据量分布不平衡。大的有数T，小的只有数百G。 我们的Spark任务，主要就是对这些数据数据进行处理，理论上，一个小时的数据起一个Spark任务。所有任务串行。这个任务串，我们称之一个任务list。

### 业务的主要逻辑

#### Step1

对原始数据，根据需求提取数据，并参照维表进行Mapping操作。例如 原始记录为15个字段。提取了其中的OriginField1，OriginField2，OriginField3 3个字段，并对OriginField2字段进行Mapping。

映射维表类似如下格式

```
a1 -> A
a2 -> A
b1 -> B
b2 -> B
b3 -> B
c1 -> C
```

这一步的目的是，通过维表对原始数据进行一个收缩。并进行归并。 如 原始数据为

```
key1,a1,1
key1,a2,1
key2,a1,2
key2,b1,3
```

Mapping 之后的结果为

```
key1,A,1
key1,A,1
key2,A,2
key2,B,3
```

再以(OriginField1,OriginField2)为key，进行reduce操作，作为输出。

```
key1,A,2
key2,A,2
key2,B,3
```

#### Step2

对Mapping并进行然后根据Mapping后的结果，按天段进行聚合，进一步缩小数据量。

#### Step3

对按天聚合的结果，注册为临时表，使用SparkSQL 进行取数操作。

### 资源情况

使用Yarn FAIR调度，一共分配给我们1000 vcore，4000g内存 的计算资源。且没有开启动态调整。这个是硬性规定，我们无法调整。

### 运行情况概述

每天只能在固定时间，通过crontab调脚本，起任务。每天起两个任务理想情况，当天起的任务能在48小时内结束，也就是说，大部分情况，整个集群会存在2个任务list并行的情况。

```
day1	day2	day3	day4
day1的任务			
	day2的任务		
		day3的任务	
```

## 问题发现与解决

### 问题出现

为描述方便，将连续4天定义为day1，day2，day3，day4。 正常情况，在day3的任务list启动时，day1的任务list应该已经结束。但实际情况是，在day3的任务list启动时，day1的任务list还在运行。并且直到day4任务list启动时，还没有结束。也就是同时存在4个任务list并行的情况。

### 问题分析

当问题出现之后，便协调我们开发进行排查。但是非常尴尬的是，没有任何以往的情况，可以用来分析。 （这里不得不吐槽下我们的运维，真的是恨不得什么都开发写完，他只用敲个sh run.sh，当然我没有任何看不起运维这个职业的意思，docker 也是一群运维大神搞的，devops 也是很棒的一个概念。我想吐槽的只是那些只知道sh run.sh 的运维。）

### 收集数据

既然没有数据，那第一步就先开始收集数据。我们的任务都是yarn-cluster模式运行的。Spark日志需要另外申请，能获取到的，也只有nohup.out中的日志，也就是如下形式。

```
18/08/20 11:56:25 INFO impl.YarnClientImpl: Submitted application application_1534428021940_5843
18/08/20 11:56:26 INFO yarn.Client: Application report for application_1534428021940_5843 (state: ACCEPTED)
18/08/20 11:56:26 INFO yarn.Client:
         client token: N/A
         diagnostics: N/A
         ApplicationMaster host: N/A
         ApplicationMaster RPC port: -1
         queue: root.bigdata_dev_dashuju
         start time: 1534737385732
         final status: UNDEFINED
         tracking URL: http://namenode3:8088/proxy/application_1534428021940_5843/
         user: bugeinikan
18/08/20 11:56:27 INFO yarn.Client: Application report for application_1534428021940_5843 (state: ACCEPTED)
18/08/20 11:56:28 INFO yarn.Client: Application report for application_1534428021940_5843 (state: ACCEPTED)
18/08/20 11:56:29 INFO yarn.Client: Application report for application_1534428021940_5843 (state: RUNNING)
18/08/20 11:56:29 INFO yarn.Client:
         client token: N/A
         diagnostics: N/A
         ApplicationMaster host: 192.168.111.161
         ApplicationMaster RPC port: 0
         queue: root.bigdata_dev_dashuju
         start time: 1534737385732
         final status: UNDEFINED
         tracking URL: http://namenode3:8088/proxy/application_1534428021940_5843/
         user: bugeinikan
18/08/20 11:56:30 INFO yarn.Client: Application report for application_1534428021940_5843 (state: RUNNING)
18/08/20 11:56:31 INFO yarn.Client: Application report for application_1534428021940_5843 (state: RUNNING)
.
.
.
18/08/20 11:57:05 INFO yarn.Client: Application report for application_1534428021940_5843 (state: FINISHED)
```

由于我们并不是要解决Spark程序报错，只是优化调度，这个日志对我们而言，足够了。 这个日志中包含了很多重要的信息：任务的submit时刻，任务accepted的时刻，任务开始running的时刻，任务finished的时刻。 由这四个时刻，可以计算出任务的submit，accepted，running三个部分的耗时。 写个代码，遍历了一遍日志就获取到了我想要的信息。

```
date	avg_submittime	avg_accepttim	avg_runningtime
20180720	9.41 	278.67 	784.11 
20180721	9.62 	298.81 	643.48 
20180722	9.81 	269.29 	674.23 
20180723	9.09 	60.82 	563.43 
20180724	10.39 	41.66 	654.18 
20180725	13.59 	22.77 	624.83 
```

#### 这里说明下Spark中，这几个时间的含义：

* SUBMIT 耗时：Spark任务提交到集群的时候，存在环境变量检查，jvm启动等过程，感兴趣的可以看这篇文章[Spark源码解析之任务提交（spark-submit）篇](https://blog.csdn.net/do\_yourself\_go\_on/article/details/75005204)。总结一下，就是这部分耗时基本不变，且难以优化。（除非减少任务数量）
* ACCEPTED 耗时：任务提交成功之后，便会进入accepted状态，如果集群资源足够，那么就可以马上分配资源进如RUNNING状态，如果资源存在竞争，申请不到启动任务所需的最小资源数，就处于等待状态。可以看到，我们的任务存在非常严重资源竞争情况，等待时间过长。
* RUNNING 耗时：这部分就是任务运行耗时了。

### 数据分析

从上一节收集的数据，可以发现我们任务慢的一个原因就是等待时间过长。也就是资源竞争造成的。我们之前的预估，是给每个任务list 500个vcore的计算资源，按理来说，不应出现竞争情况。于是便找运维要来了提交脚本。很快就发现了原因。 提交脚本部分如下：

```
spark-submit --master yarn-cluster --name XXX --driver-memory 6g --executor-memory 6g  --num-executors 500
```

好吧，每个executor分了6g 的内存，500 \* 6g \* 2 = 6000g 已经超过了4000g的分配资源，自然存在竞争。（好吧，以后这个run.sh还是我们来写吧 XD）

### 改进及前后数据对比

所以，我们做的修改，只是修改了一个提交参数。然后任务list就恢复了正常。 后来也问了运维为啥给6g，答曰：习惯。。。。。。

### 进一步的优化

本着精益求精的方式，当然需要谋求进一步的优化。因为各个数据源中，数据量的不同，存在这样的情况，有的小时文件片数只有300片，如果分配500个executor，根本无法利用满这500个executor。 因而通过统计不同数据源的文件片数，将任务list进行拆分，将每小时文件片数在200片以下的放到一个队列，将每小时文件片数在200以上的放到另一个队列。提高了资源的利用率，使得当天启动的任务list能够在24小时内跑完。调整之后，平均每片文件的运行时间如下：

```
date	avg_submittime	avg_accepttim	avg_runningtime	avg_files	avg_timeperfile
20180720	9.41 	278.67 	784.11 	1,227.05 	0.81 
20180721	9.62 	298.81 	643.48 	1,076.81 	0.78 
20180722	9.81 	269.29 	674.23 	1,283.50 	0.56 
20180723	9.09 	60.82 	563.43 	1,054.95 	0.62 
20180724	10.39 	41.66 	654.18 	1,103.82 	0.69 
20180725	13.59 	22.77 	624.83 	1,202.89 	0.60 
20180726	14.90 	25.48 	415.99 	1,031.74 	0.61 
20180727	9.82 	17.07 	331.34 	1,118.17 	0.45 
```

## 总结

记录了一次Spark任务调度调优的过程。 想要调优，必须先收集数据，基于数据说话，才能更好的发现问题，与定位问题。
