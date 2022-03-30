# Spark Coding技巧

## 嵌套格式读取

```
x.getAs[Seq[Row]](5).map(g=>{(g.getString(1),g.getString(2))})
```

## dataFrame 中的vector 转为 array

```
val rt = rdd_train.select("selected_features").rdd.map(x=>x(0).asInstanceOf[org.apache.spark.mllib.linalg.DenseVector])
```

## spark 读取Avro

```
val dataIn = "/data/etl/fetch/666_dpi_result/p_biz=game/p_date=201807{18,19,20,21,22,23,24}/*844*.avro"
sqlContext.read.format("com.databricks.spark.avro").load(dataIn).registerTempTable("t_game")
val sql1 = "select phone_number, log_date, hf from t_game  LATERAL VIEW explode(host_freq) adTable  as hf"
val df = sqlContext.sql(sql1)
```

可以使用通配符读取avro文件，但是存在Schema不一致时，会报错

## spark textFile加载多个目录：

其实很简单，将多个目录（对应多个字符串），用,作为分隔符连接起来

```
val inputPath = List("hdfs://localhost:9000/test/hiveTest", "hdfs://localhost:9000/test/hiveTest2")
                    .mkString(",")
sparkContext
      .textFile(inputPath)
```

## 获取时间

```
val iterationStartTime = System.nanoTime()
```

这种写法，值得学习

## spark脚本日志输出级别设置

```
import org.apache.log4j.{ Level, Logger }

Logger.getLogger("org").setLevel(Level.WARN)
Logger.getLogger("org.apache.spark").setLevel(Level.WARN)
Logger.getLogger("org.eclipse.jetty.server").setLevel(Level.WARN)

spark.sparkContext.setLogLevel("WARN")
```

## rdd.saveAsTextFile 输出到HDFS 文件压缩

```
rdd.saveAsTextFile( "hdfs://localhost:9000/test/out" ) //正常不压缩
rdd.saveAsTextFile( "hdfs://localhost:9000/test/outGzip", classOf[ GzipCodec ] )    //Gzip压缩输出
rdd.saveAsTextFile( "hdfs://localhost:9000/test/outBzip2", classOf[ BZip2Codec ] )  //bzip2 压缩输出
```

[hadoop 文件压缩格式对比](http://www.echojb.com/web-application-server/2017/07/10/449381.html)

## Spark 2.x 之后，设置num-executors 数据

Spark 1.6 --num-executors NUM Number of executors to launch (Default: 2).

Spark 2.x --num-executors NUM Number of executors to launch (Default: 2). If dynamic allocation is enabled, the initial number of executors will be at least NUM.

## pySpark开启日志

做如下设置

```
sc.setLogLevel("FATAL")
```

[pySpark开启日志](https://stackoverflow.com/questions/25193488/how-to-turn-off-info-logging-in-spark)

## 本地调试

Spark 使用intellijidea本地调试

1. 添加spark-assembly-1.3.1-hadoop.jar
2. 点击run-》edit configurations,点击左上角绿色加号，添加application，mainclass选择为即将调试的class，vm options填写：-Dspark.master=local，点击OK结束
3. 注意1.3对应scala版本，否则出现\*\*\*\*\*hashset等类似错误

## 将pySpark 的代码迁移至scala

注意 元组下标和数组下标

## Scala使用正则表达式

匹配特定起始标记与特定结束标记之间的子串，正则式子要重头写到尾，在想要的部分上加上括号：

```
val in = "/data/resource/public960/mapping_deviceid/831.txt"
val pattern = """.*/([0-9]{3})\.txt""".r
val b = in match{
  case pattern(n) => n
}
println(b)
```

[ref](http://blog.csdn.net/o1101574955/article/details/52551110)
