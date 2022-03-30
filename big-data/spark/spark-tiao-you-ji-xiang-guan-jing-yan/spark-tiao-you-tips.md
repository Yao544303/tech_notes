# Spark 调优Tips

## 优化GC参数

spark2-submit --master yarn --deploy-mode cluster --name pi --executor-cores 2 --executor-memory 1g --driver-memory 6g --driver-cores 6 --num-executors 5 --class com.xxx.testSpark original-spark2-1.0.jar 10000 --conf "spark.driver.extraJavaOptions=-XX:+PrintFlagsFinal -XX:+PrintReferenceGC -verbose:gc -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintAdaptiveSizePolicy -XX:+UnlockDiagnosticVMOptions -XX:+G1SummarizeConcMark"

## task数量设置

task数量，设置成spark application总cpu core数量的2\~3倍 如何设置一个Spark Application的并行度？

```
spark.default.parallelism
SparkConf conf = new SparkConf()
conf.set("spark.default.parallelism", "500")
```

[ref](http://blog.csdn.net/hutao\_hadoop/article/details/52693856)

## filter 之后，考虑reparation

## split 之后，注意filter ，去掉不符合要求的数据

## saveAsText 前记得 coalesce ，防止小文件过多

## 字符串拼接尽量不要使用 a + b 这种方式，使用占位符

```
print("the accesstag count is : %i" % accesstagct)
print("the sendtag count is : %i" % sendtagct)
print("the bank count is :")
for (bk, ct) in bank:
    print("%s: %i" % (bk, ct))
```

## 在大规模数据操作时，reduceByKey 优于 groupByKey

[reduceByKey 优于 groupByKey](http://blog.csdn.net/zongzhiyuan/article/details/49965021)

## map侧 join

## 广播变量

## 组成一个list 来代替 groupby

## 一些新入门的人会遇到搞不清transform和action，没有明白transform是lazy的，需要action触发，并且两个action前后调用效果可能不一样。

## 实时处理方面：一方面要注意数据源（Kafka）topic需要多个partition，并且数据要散列均匀，使得Spark Streaming的Recevier能够多个并行，并且均衡地消费数据 。使用Spark Streaming，要多通过Spark History 排查DStream的操作中哪些处理慢，然后进行优化。另外一方面我们自己还做了实时处理的监控系统，用来监控处理情况如流 入、流出数据速度等。通过监控系统报警,能够方便地运维Spark Streaming 实时处理程序。这个小监控系统主要用了 influxdb+grafana 等实现。

## 大家使用过程当中，对需要重复使用的RDD，一定要做cache，性能提升会很明显。
