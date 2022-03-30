# Spark常见问题

## 依赖冲突报错

java.lang.IncompatibleClassChangeError: class com.databricks.spark.avro.AvroRelation has interface org.apache.spark.sql.sources.TableScan as super class 调整版本

## 字符串注意

python 中读取文件，如果字段为int64 类型，会自动转换，但是 spark 中，textfile 读取的都是string。 然后 就会出现 我觉得1 +2 = 3，但实际上是1 + 2 = 12 这种奇葩的现象了

## 字符串分割 "|" 和 '|' 差距很大

分割注意空的

```
a|a||||
```

这个分割就a a ,后面都是为空 可以如下操作

```
.map(_+"|A")
.map(_.split('|'))
.map(_.split("\\|", -1))
```

[String.split("","")与StringUtil.split("","")的区别](http://blog.csdn.net/yayayaya\_\_/article/details/37689861)

## java.lang.OutOfMemoryError: GC overhead limit exceeded

两种方法是，增加参数，-XX:-UseGCOverheadLimit，关闭这个特性，同时增加heap大小，-Xmx1024m。 [ref](https://www.cnblogs.com/hucn/p/3572384.html)

## ERROR executor.CoarseGrainedExecutorBackend: RECEIVED SIGNAL 15: SIGTERM

出现这个错误，基本就是oom了[REF](https://www.cnblogs.com/fbiswt/p/4679403.html)

## Kryo serialization failed: Buffer overflow

1. 是调大spark.kryoserializer.buffer.max没有设置对. 最大可以设置为2048mb.
2. 是在SparkSQL 中，使用小表去join大表，好像在spark2.0 已经优化了 [Kryo](http://blog.csdn.net/edin\_blackpoint/article/details/72783747)

## SparkSQL 访问hive时，使用sqlContext 而非hiveContext报错

```
java.lang.RuntimeException: [1.1] failure: ``with'' expected but identifier use found
use bigdata_dev_dashuju
```

使用hiveContext即可

```
val sqlContext = new org.apache.spark.sql.hive.HiveContext(sc)
```

## Maven 编译报错\[ERROR] error: error while loading Entry, class file 'C:\Program Files\Java\jdk1.8.0\_77\jre\lib\rt.jar(java/util/Map$Entry.class)' is broken

pom 中需要要完整的scala 版本 [REF](http://blog.csdn.net/u011098327/article/details/74529099)

## 记一次spark mllib stackoverflow踩坑

spark在迭代计算的过程中，会导致linage剧烈变长，所需的栈空间也急剧上升，最终爆栈了。。 [REF](https://blog.csdn.net/asdfghjkl1993/article/details/78626439)

## sqlContext.load 输入异常

sqlContext.read.format("com.databricks.spark.avro").load(in).registerTempTable("t\_table")

### in 不存在时，报错

java.io.FileNotFoundException: The path (hdfs://nameservice/group/bigdata\_dev\_dashuju/files/666\_test/dtDayNotExist) is invalid.

### in 为空时 报错

java.io.FileNotFoundException: No avro files present at hdfs://nameservice/group/bigdata\_dev\_dashuju/files/gom/666\_test/testCases/histroyIsNull

## textFile 输入异常

### in 不存在时

读取时不报错，执行action算子时报错

```
org.apache.hadoop.mapred.InvalidInputException: Input path does not exist: hdfs://nameservice/group/bigdata_dev_dashuju/files/666_test/dtDayNotExist
故而，最好加一层判断
```

### in 为空时

都不报错

## pyspark 的文件名 不要使用select 等字符

比如select.py 会报错

## Spark 双主机启动不能

主要由于缓存的问题，会出现脑裂

1. 关闭各个服务器上的Spark进程

```
	$ stop-all.sh 
	$ stop-master.sh 
	这两个不一定能有用的情况下，可以通过JPS查询master 和 worker的进程号，然后手动kill
```

1. 分别在两台master上启动

```
 在主上启动 $ start-all.sh
 在备上启动 $ start-master.sh
```

## 启动spark shell 报错，无法读取csv 文件

spark-shell --packages com.databricks:spark-csv\_2.10:1.3.0

## Spark-SQl 启动错误，找不到jdbc包之类 需要另外导入包

## Spark shell 中job变量报错

[解决Spark shell模式下初始化Job出现的异常](https://www.iteblog.com/archives/2142.html)

## 我们测试网经常出现找不到第三方jar的情况，如果是用CDH的同学一般会遇到，就是在CDH 5.4开始，CDH的技术支持人员说他们去掉了hbase等一些jar，他们认那些jar已经不需要耦合在自己的classpath中，这个情况可以通过spark.executor.extraClassPath方式添加进来。

## 序列化报错问题

如下代码

```
      val rdd = sc.hadoopFile[AvroWrapper[GenericRecord], NullWritable, AvroInputFormat[GenericRecord]](filePath)
        .map(_._1.datum())
        .take(10).foreach(println)
```

如上，这个会有序列化问题

```
      val rdd = sc.hadoopFile[AvroWrapper[GenericRecord], NullWritable, AvroInputFormat[GenericRecord]](filePath)
        .map(_._1.datum().toString)
        .take(10).foreach(println)
```

如上，这个就不会有序列化问题，toString了

## 在RDD中非序列化变量的操作

要小心一些非序列化的变量 如一些类的get set 方法之后，必须转为可序列化的对象，toString 并且在序列化后，不可reparation

## Spark OOM：java heap space，OOM:GC overhead limit exceeded解决方法

问题描述:在使用Spark过程中，有时会因为数据增大，而出现下面两种错误:

* Java.lang.OutOfMemoryError: Java heap space
* java.lang.OutOfMemoryError：GC overhead limit exceeded

这两种错误之前我一直认为是executor的内存给的不够，但是仔细分析发现其实并不是executor内存给的不足，而是driver的内存给的不足。在standalone client模式下用spark-submit提交任务时（standalone模式部署时，默认使用的就是standalone client模式提交任务），我们自己写的程序（main）被称为driver，在不指定给driver分配内存时，默认分配的是512M。在这种情况下，如果处理的数据或者加载的数据很大（我是从Hive中加载数据），driver就可能会爆内存，出现上面的OOM错误。 在spark\_home/conf/目录中，将spark-defaults.conf.template模板文件拷贝一份到/spark\_home/conf目录下，命名为spark-defaults.conf，然后在里面设置spark.driver.memory memSize属性来改变driver内存大小。

## “Caused by: MetaException(message:Version information not found in metastore. )”

修改conf/hive-site.xml 中的 “hive.metastore.schema.verification” 值为 false 即可解决
