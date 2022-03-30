# Spark\_dataframe操作集合

## spark dataframe 操作集合

### 读取

```
val df = spark.read.json("jsonFile")

val usersDF = spark.read.load("examples/src/main/resources/users.parquet")

val peopleDF = spark.read.format("json").load("examples/src/main/resources/people.json")

val peopleDFCsv = spark.read.format("csv")
  .option("sep", ";")
  .option("inferSchema", "true")
  .option("header", "true")
  .load("examples/src/main/resources/people.csv")

usersDF.write.format("orc")
  .option("orc.bloom.filter.columns", "favorite_color")
  .option("orc.dictionary.key.threshold", "1.0")
  .option("orc.column.encoding.direct", "name")
  .save("users_with_options.orc")
```

Untyped Dataset Operations: 自动类型推导（如Int，String)等

### 写

```
rdd.saveAsTextFile("out")
df.write.option("header","true").csv("out")

usersDF
  .write
  .partitionBy("favorite_color")
  .bucketBy(42, "name")
  .saveAsTable("users_partitioned_bucketed")

usersDF.write.partitionBy("favorite_color").format("parquet").save("namesPartByColor.parquet")  
```

### 全局视图

之前版本 temporary view 是跟着sparksession 走的，使用全局视图，所有session 都可以用。

```
df.createGlobalTempView("people")
spark.newSession().sql("SELECT * FROM global_temp.people").show()
```

### 算子

#### show

```
show()//默认显示20行，字段值超过20，默认截断
show(numRows: Int)//指定显示行数
show(truncate: Boolean)//指定是否截断超过20的字符
show(numRows: Int, truncate: Boolean)//指定行数，和是否截断
show(numRows: Int, truncate: Int)//指定行数，和截断的字符长度
```

#### collect & collectAsList

```
scala> df.collect
res15: Array[org.apache.spark.sql.Row] = Array([null,Michael], [30,Andy], [19,Justin])

scala> df.collectAsList
res16: java.util.List[org.apache.spark.sql.Row] = [[null,Michael], [30,Andy], [19,Justin]]
```

#### decribe

获取指定字段的统计信息，结果仍为DataFrame 对象

```
jdbcDF .describe("c1" , "c2", "c4" ).show()
```

#### first, head, take, takeAsList

获取若干行记录 这里列出的四个方法比较类似，其中

* first获取第一行记录
* head获取第一行记录，head(n: Int)获取前n行记录
* take(n: Int)获取前n行数据
* takeAsList(n: Int)获取前n行数据，并以List的形式展现 　　以Row或者Array\[Row]的形式返回一行或多行数据。first和head功能相同。 　　take和takeAsList方法会将获得到的数据返回到Driver端，所以，使用这两个方法时需要注意数据量，以免Driver发生OutOfMemoryError

#### filter && where

* where 调用的是filter
* where(conditionExpr: String)：SQL语言中where关键字后的条件传入筛选条件表达式，可以用and和or。得到DataFrame类型的返回结果，

```
df.where("id = 1 or c1 = 'b'").show()
```

* filter: 根据字段进行筛选，传入筛选条件表达式，得到DataFrame类型的返回结果。和where使用条件相同

```
df.filter("id = 1 or c1 = 'b'").show()
```

#### select 相关

* select：获取指定字段值，根据传入的String类型字段名，获取指定字段的值，以DataFrame类型返回

```
jdbcDF.select( "id" , "c3" ).show( false)

// 还有一个重载的select方法，不是传入String类型参数，而是传入Column类型参数。可以实现select id, id+1 from test这种逻辑。
jdbcDF.select(jdbcDF( "id" ), jdbcDF( "id") + 1 ).show( false)
```

* selectExpr：可以对指定字段进行特殊处理，可以直接对指定字段调用UDF函数，或者指定别名等。传入String类型参数，得到DataFrame对象。 　　示例，查询id字段，c3字段取别名time，c4字段四舍五入：

```
jdbcDF .selectExpr("id" , "c3 as time" , "round(c4)" ).show(false)
```

* col：获取指定字段，只能获取一个字段，返回对象为Column类型。

```
val idCol = jdbcDF.col(“id”)
```

* apply：获取指定字段，只能获取一个字段，返回对象为Column类型

```
val idCol1 = jdbcDF.apply("id")
val idCol2 = jdbcDF("id")
```

* drop：去除指定字段，保留其他字段,返回一个新的DataFrame对象，其中不包含去除的字段，一次只能去除一个字段。 　　示例：

```
jdbcDF.drop("id")
jdbcDF.drop(jdbcDF("id"))
```

#### limit

limit方法获取指定DataFrame的前n行记录，得到一个新的DataFrame对象。和take与head不同的是，limit方法不是Action操作。

```
jdbcDF.limit(3).show()
```

#### order by

orderBy和sort：按指定字段排序，默认为升序

* 按指定字段排序。加个-表示降序排序。sort和orderBy使用方法相同

```
jdbcDF.orderBy(- jdbcDF("c4")).show(false)
// 或者
jdbcDF.orderBy(jdbcDF("c4").desc).show(false)
```

* 按字段字符串升序排序

```
jdbcDF.orderBy("c4").show(false)
```

* sortWithinPartitions 和上面的sort方法功能类似，区别在于sortWithinPartitions方法返回的是按Partition排好序的DataFrame对象。

#### 添加自增列

```
    import org.apache.spark.sql.functions._
    val inputDF = inputDF.withColumn("id", monotonically_increasing_id)
```

#### group by

* groupBy：根据字段进行group by操作,　groupBy方法有两种调用方式，可以传入String类型的字段名，也可传入Column类型的对象。 　　使用方法如下

```
jdbcDF .groupBy("c1" )
jdbcDF.groupBy( jdbcDF( "c1"))
```

* cube和rollup：group by的扩展，功能类似于SQL中的group by cube/rollup，
* GroupedData对象，该方法得到的是GroupedData类型对象，在GroupedData的API中提供了group by之后的操作，比如，

```
    max(colNames: String*)方法，获取分组中指定字段或者所有的数字类型字段的最大值，只能作用于数字型字段
    min(colNames: String*)方法，获取分组中指定字段或者所有的数字类型字段的最小值，只能作用于数字型字段
    mean(colNames: String*)方法，获取分组中指定字段或者所有的数字类型字段的平均值，只能作用于数字型字段
    sum(colNames: String*)方法，获取分组中指定字段或者所有的数字类型字段的和值，只能作用于数字型字段

    count()方法，获取分组中的元素个数
```

```
　
```

* agg，pivot 该方法和下面介绍的类似，可以用于对指定字段进行聚合操作。

```
df.groupBy("uid").agg(size(collect_set("host")), size(collect_set("date")))
```

#### distinct

* distinct：返回一个不包含重复记录的DataFrame，返回当前DataFrame中不重复的Row记录。该方法和接下来的dropDuplicates()方法不传入指定字段时的结果相同。

```
jdbcDF.distinct()
```

* dropDuplicates：根据指定字段去重,根据指定字段去重。类似于select distinct a, b操作

```
jdbcDF.dropDuplicates(Seq("c1"))
```

#### 聚合

聚合操作调用的是agg方法，该方法有多种调用方式。一般与groupBy方法配合使用。　　以下示例其中最简单直观的一种用法，对id字段求最大值，对c4字段求和。

```
jdbcDF.agg("id" -> "max", "c4" -> "sum")
```

#### union

* unionAll方法：对两个DataFrame进行组合。类似于SQL中的UNION ALL操作。

```
jdbcDF.unionALL(jdbcDF.limit(1))
```

#### join

重点来了。在SQL语言中用得很多的就是join操作，DataFrame中同样也提供了join的功能。　接下来隆重介绍join方法。在DataFrame中提供了六个重载的join方法。

* 笛卡尔积

```
joinDF1.join(joinDF2)
```

* using一个字段形式,下面这种join类似于a join b using column1的形式，需要两个DataFrame中有相同的一个列名，joinDF1和joinDF2根据字段id进行join操作，结果如下，using字段只显示一次。

```
joinDF1.join(joinDF2, "id")
```

* using多个字段形式，除了上面这种using一个字段的情况外，还可以using多个字段，如下

```
joinDF1.join(joinDF2, Seq("id", "name")）
```

* 指定join类型,两个DataFrame的join操作有inner, outer, left\_outer, right\_outer, leftsemi类型。在上面的using多个字段的join情况下，可以写第三个String类型参数，指定join的类型，如下所示

```
joinDF1.join(joinDF2, Seq("id", "name"), "inner"）
```

* 使用Column类型来join,如果不用using模式，灵活指定join字段的话，可以使用如下形式

```
joinDF1.join(joinDF2 , joinDF1("id" ) === joinDF2( "t1_id"))
```

* 在指定join字段同时指定join类型

```
joinDF1.join(joinDF2 , joinDF1("id" ) === joinDF2( "t1_id"), "inner")
```

* not in

```
df1.join(df_seed_negtive,"uid","leftanti")
```

#### 获取指定字段统计信息

stat方法可以用于计算指定字段或指定字段之间的统计信息，比如方差，协方差等。这个方法返回一个DataFramesStatFunctions类型对象。 下面代码演示根据c4字段，统计该字段值出现频率在30%以上的内容。在jdbcDF中字段c1的内容为"a, b, a, c, d, b"。其中a和b出现的频率为2 / 6，大于0.3

```
jdbcDF.stat.freqItems(Seq ("c1") , 0.3).show()
```

#### 获取两个DataFrame中共有的记录

intersect方法可以计算出两个DataFrame中相同的记录，

```
jdbcDF.intersect(jdbcDF.limit(1)).show(false)
```

#### 获取一个DataFrame中有另一个DataFrame中没有的记录

```
jdbcDF.except(jdbcDF.limit(1)).show(false)
```

#### 操作字段名

* withColumnRenamed：重命名DataFrame中的指定字段名,如果指定的字段名不存在，不进行任何操作。下面示例中将jdbcDF中的id字段重命名为idx。

```
jdbcDF.withColumnRenamed( "id" , "idx" )
```

* withColumn：往当前DataFrame中新增一列,whtiColumn(colName: String , col: Column)方法根据指定colName往DataFrame中新增一列，如果colName已存在，则会覆盖当前列。以下代码往jdbcDF中新增一个名为id2的列，

```
jdbcDF.withColumn("id2", jdbcDF("id")).show( false)
```

#### 调整数据类型

```
import org.apache.spark.sql.functions.col
import org.apache.spark.sql.types.IntegerType

// Convert String to Integer Type
df.withColumn("salary",col("salary").cast(IntegerType))
df.withColumn("salary",col("salary").cast("int"))
df.withColumn("salary",col("salary").cast("integer"))

// Using select
df.select(col("salary").cast("int").as("salary"))

//Using selectExpr()
df.selectExpr("cast(salary as int) salary","isGraduated")
df.selectExpr("INT(salary)","isGraduated")

//Using with spark.sql()
spark.sql("SELECT INT(salary),BOOLEAN(isGraduated),gender from CastExample")
spark.sql("SELECT cast(salary as int) salary, BOOLEAN(isGraduated),gender from CastExample")
```

#### 行转列

有时候需要根据某个字段内容进行分割，然后生成多行，这时可以使用explode方法，下面代码中，根据c3字段中的空格将字段内容进行分割，分割的内容存储在新的字段c3\_中，如下所示

```
jdbcDF.explode( "c3" , "c3_" ){time: String => time.split( " " )}

//spark 2.2+
val service_result = df_info.withColumn("service_explode", explode_outer(col("service_prod_code")))

//spark <=2.1
df.withColumn("service_explode", explode(
  when(col("service_prod_code").isNotNull, col("service_prod_code"))
    // If null explode an array<string> with a single null
    .otherwise(array(lit(null).cast("string")))))
    
// pyspark 版本
from pyspark.sql.functions import explode
df1 = df.withColumn("host_arr",explode("hosts"))
```

#### 列转行

```
    val rowRDD = pivotRDD
      .map(_.split("\t"))
      .map(attributes => Row(attributes(0), attributes(1), attributes(2), attributes(3).toInt))
    val peopleDF = spark.createDataFrame(rowRDD, schema)
    peopleDF.createOrReplaceTempView("pivot")
 
    val results = spark.sql("SELECT * FROM pivot")
    results.show()
    results.groupBy("A", "B").pivot("C").sum("D").show()
    
    val df = spark.sqlContext.createDataFrame(rdd).toDF("host","type","count").groupBy("host").pivot("type").sum("count")
    
    df.select("uid","host","province")
    .distinct.groupBy("province","host").count
    .groupBy("host")
    .pivot("province")
    .sum("count")
    // provice,host,count -> host,812
```

#### na()

返回包含null值的行

```
val df=spark.read.json("C:\\zlq\\data\\people.json")
df.show()
df.na.drop().show()
```

#### stat()

```
//{"name":"Michael"}
//{"name":"Andy", "age":30}
//{"name":"Justin", "age":19}
//{"name":"Tom", "age":19}
//{"name":"jerry", "age":19}
df.stat.freqItems(Seq("age")).show()
```

#### UDF

```
      import org.apache.spark.sql.functions._
      def getRn(op: String, nn:String): String ={
        var re = ""
        if(op == "1") { re = s"888_${nn}"}
        if(op == "0") { re = s"666_${nn}"}
        if(op == "2") { re = s"999_${nn}"}
        return re
      }
      spark.udf.register("getRn", getRn _)

      val (today,dlist) = getRestDay(1)
      val zTmp = "/user/yaocz/z_projcet/zs_plan_tmp"
      val output = s"/user/yaocz/yx_prepare/${today}"
      val dft = spark.read.csv(zTmp).toDF("uid","rn")
      val dfMap = spark.read.format("parquet").load("/dw/cdm/dim/mapping_uid_property")
      dft.join(dfMap,"uid").selectExpr("uid","operator","getRn(operator,rn)",s"'${today}'")
        .write.csv(output)
```

### REF

* [Spark-SQL之DataFrame操作大全](https://blog.csdn.net/dabokele/article/details/52802150)
* [spark sql Dataset\&Dataframe算子大全](https://blog.csdn.net/zhaolq1024/article/details/88699916#RDD%2CDataset%2CDataframe%E4%BA%92%E7%9B%B8%E8%BD%AC%E6%8D%A2)
