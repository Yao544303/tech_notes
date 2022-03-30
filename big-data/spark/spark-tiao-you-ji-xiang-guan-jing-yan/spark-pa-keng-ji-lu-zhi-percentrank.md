# Spark爬坑记录之percent\_rank

在Kmeans 预处理之前的数据预处理部分，需要做一次类似percent\_rank() 操作。 举个例子，输入一条记录为user\_1,key1,value\_1 输入多条记录 按照key1 聚合之后，结果为key1,list\[value1,value2.....] 将list 排序，并获取count 结果为 key1，count，list 然后计算user1 这条记录的rank = index（value1）/count

```
    val ah_freqlist_dic_old = user_ah_freq_dic.map(x=>(x._1._2,Nil:+x._2))
      .reduceByKey(_:::_)
      .map(x=>(x._1,(x._2.length).toString +: x._2.sorted))
      .collectAsMap()
val ah_freqlist_dic = sc.broadcast(ah_freqlist_dic_old)
val user_ah_percent_dic = user_ah_freq_dic.map(t=>((t._1._1,t._1._2),ah_freqlist_dic.value.getOrElse(t._1._2,Nil).indexOf(t._2)/((ah_freqlist_dic.value.getOrElse(t._1._2,Nil)(0).toInt).toFloat)))
```

然后效率极低。

当把这段代码换成 percent\_rank() 之后，奇迹发生了

```
 val df = sqlContext.createDataFrame(user_ah_freq_dic).toDF("user","ah","freq")
    df.registerTempTable("uahf")
    val sql="select user,ah,percent_rank() over (partition by ah order by freq) as rank from uahf"

    val user_ah_percent_dic = sqlContext.sql(sql).rdd.map(x=>((x(0).toString,x(1).toString),x(2).toString.toFloat))
```

果然spark sql 的是做了优化的。阅读spark sql 的源码，又有计划了
