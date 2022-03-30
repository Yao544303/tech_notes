# Spark UDF

## udf函数的 两种注册方式：udf() 和 register()

* spark.udf.register() // spark是SparkSession对象
* udf() // 需要import org.apache.spark.sql.functions.

## 在dataframe中使用

```
// 定义自定义函数
def add_one(col: Double) = {
      col + 1
    }

// 注册自定义函数
spark.udf.register("add_one", add_one _)

// 使用自定义函数
import org.apache.spark.sql.functions
dataframe.withColumn("a2", functions.callUDF("add_one", functions.col("a")))
```

## 在sparkSQL中使用

```
// 定义自定义函数
def strLen(col: String) = {
    str.length()
}

// 注册自定义函数
spark.udf.register("strLen", strLen _)

// 使用自定义函数
spark.sql("select name,strLen(name) from table ")
```

## SparkSQL中UDF功能的权限控制机制是怎样的？

目前已有的SQL语句无法满足用户场景时，用户可使用UDF功能进行自定义操作。 为确保数据安全以及UDF中的恶意代码对系统造成破坏，SparkSQL的UDF功能只允许具备admin权限的用户注册，由admin用户保证自定义的函数的安全性。

## REF

* [在spark中使用UDF函数](https://zhuanlan.zhihu.com/p/64410979)
* [SparkSQL UDF功能的权限控制机制](https://support.huaweicloud.com/devg-mrs/mrs\_06\_0252.html)
