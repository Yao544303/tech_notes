# 我为什么从Spark又回到了MapReduce

## 简介

近年来，随着Spark的处理能力不断增强，技术栈覆盖面愈发广泛，社区活跃度感人，渐有取代MapRedece之势，但Spark真的就全方位吊打MapReduce么？ 笔者将结合自己最近在工作中

## Spark简介

Apache Spark 是专为大规模数据处理而设计的快速通用的计算引擎。拥有Hadoop MapReduce所具有的优点；但不同于MapReduce的是——Job中间输出结果可以保存在内存中，从而不再需要读写HDFS，因此Spark能更好地适用于数据挖掘与机器学习等需要迭代的MapReduce的算法。

## MapReduce 简介

Map/Reduce是在2004年谷歌的一篇论文中提出大数据并行编程框架，由两个基本的步骤Map(映射)和Reduce(化简)组成，Map/Reduce由此得名。同时，由于它隐藏了分布式计算中并行化、容错、数据分布、负载均衡等内部细节，实际的使用中普通编程人员/应用人员只需关系map/reduce两个过程的实现，大大的降低了实现分布式计算的要求，因此在日志分析、海量数据排序等等场合中广泛应用。

## 稳定性

在架构方面，由于大量数据被缓存在RAM中，Java回收垃圾缓慢的情况严重，导致Spark性能不稳定，在复杂场景中SQL的性能甚至不如现有的Map/Reduce。

## 容错

和 MapReduce 一样， Spark 会重试每个任务并进行预测执行。然而，MapReduce 是依赖于硬盘驱动器的，所以如果一项处理中途失败，它可以从失败处继续执行，而 Spark 则必须从头开始执行，所以 MapReduce 这样节省了时间。

## REF

* [Spark在任何情况下均比MapReduce高效吗？](https://blog.csdn.net/sunspeedzy/article/details/69062802)
