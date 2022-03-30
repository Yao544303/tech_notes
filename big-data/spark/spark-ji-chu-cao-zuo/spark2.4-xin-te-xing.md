# Spark2.4新特性

## Spark 2.4 新特性

## [Spark SQL 查询中 Coalesce 和 Repartition 暗示（Hint）](https://www.iteblog.com/archives/2501.html)

Spark 2.4 之后可用

```
spark.sql("create table iteblog1 as select /*+ REPARTITION(4) */ age,count(*) from person where age between 10 and 20 group by age").explain()
```

## [Apache Spark 2.4 内置图像数据源介绍](https://www.iteblog.com/archives/2478.html)

## [Apache Spark 2.4 内置的 Avro 数据源介绍](https://www.iteblog.com/archives/2476.html)

从 Apache Spark 2.4 版本开始，Spark 为读取和写入 Avro 数据提供内置支持。新的内置 spark-avro 模块最初来自 Databricks 的开源项目Avro Data Source for Apache Spark。除此之外，它还提供以下功能：

* 新函数 from\_avro() 和 to\_avro() 用于在 DataFrame 中读取和写入 Avro 数据，而不仅仅是文件。
* 支持 Avro 逻辑类型（logical types），包括 Decimal，Timestamp 和 Date类型。
* 2倍读取吞吐量提高和10％写入吞吐量提升。

结论：内置的 spark-avro 模块为 Spark SQL 和 Structured Streaming 提供了更好的用户体验以及 IO 性能。

## [Apache Spark 2.4 中解决复杂数据类型的内置函数和高阶函数介绍](https://www.iteblog.com/archives/2457.html)

### 之前的方案

* Explode and Collect

```
SELECT id,
       collect_list(val + 1) AS vals
FROM   (SELECT id,
               explode(vals) AS val
        FROM iteblog) x
GROUP BY id
```

* User Defined Function

```
def addOne(values: Seq[Int]): Seq[Int] = {
  values.map(value => value + 1)
}
val plusOneInt = spark.udf.register("plusOneInt", addOne(_: Seq[Int]): Seq[Int])
```

### 新方案

* 高阶函数（Higher-Order Functions）

```
SELECT TRANSFORM(
    nested_values,
    arr -> TRANSFORM(arr,
      element -> element + key + SIZE(arr)))
FROM iteblog;
```

\########################################

## [Apache Spark 2.0 在作业完成时却花费很长时间结束](https://www.iteblog.com/archives/2500.html)

现象：虽然我们的 Spark Jobs 已经全部完成了，但是我们的程序却还在执行。比如我们使用 Spark SQL 去执行一些 SQL，这个 SQL 在最后生成了大量的文件。然后我们可以看到，这个 SQL 所有的 Spark Jobs 其实已经运行完成了，但是这个查询语句还在运行。通过日志，我们可以看到 driver 节点正在一个一个地将 tasks 生成的文件移动到最终表的目录下面，当我们作业生成的文件很多的情况下，就很容易产生这种现象。本文将给大家介绍一种方法来解决这个问题。

原因： Hadoop 2.x 的 FileOutputCommitter 实现 mapreduce.fileoutputcommitter.algorithm.version 参数的值，默认为1；如果这个参数为1，那么在 Task 完成的时候，是将 Task 临时生成的数据移到 task 的对应目录下，然后再在 commitJob 的时候移到最终作业输出目录，而这个参数,在 Hadoop 2.x 的默认值就是 1！这也就是为什么我们看到 job 完成了，但是程序还在移动数据，从而导致整个作业尚未完成，而且最后是由 Spark 的 Driver 执行 commitJob 函数的，所以执行的慢也是有到底的。

而我们可以看到，如果我们将 mapreduce.fileoutputcommitter.algorithm.version 参数的值设置为 2，那么在 commitTask 执行的时候，就会调用 mergePaths 方法直接将 Task 生成的数据从 Task 临时目录移动到程序最后生成目录。而在执行 commitJob 的时候，直接就不用移动数据了，自然会比默认的值要快很多。

Spark中： 直接在 conf/spark-defaults.conf 里面设置 spark.hadoop.mapreduce.fileoutputcommitter.algorithm.version 2，这个是全局影响的。 直接在 Spark 程序里面设置，spark.conf.set("mapreduce.fileoutputcommitter.algorithm.version", "2")，这个是作业级别的。 如果你是使用 Dataset API 写数据到 HDFS，那么你可以这么设置 dataset.write.option("mapreduce.fileoutputcommitter.algorithm.version", "2")。 不过如果你的 Hadoop 版本为 3.x，mapreduce.fileoutputcommitter.algorithm.version 参数的默认值已经设置为2了
