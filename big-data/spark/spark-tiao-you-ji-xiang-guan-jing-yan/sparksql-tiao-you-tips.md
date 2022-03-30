# SparkSQL 调优Tips

## 优化过程中常用到方法

```
// 查看查询的整个运行计划 
scala>query.queryExecution 
// 查看查询的Unresolved LogicalPlan 
scala>query.queryExecution.logical
// 查看查询的Analyzed LogicalPlan
scala>query.queryExecution.analyzed
// 查看优化后的LogicalPlan
scala>query.queryExecution.optimizedPlan
// 查看物理计划
scala>query.queryExecution.sparkPlan
// 查看RDD的转换过程
scala>query.toDebugString
```

## 使用join代替not in

分析SQL代码，其中where 条件中有NOT IN（select roleid from per..）子查询。 去掉not in子查询，进行查询，能在3s内结果，由此该SQL性能瓶颈就出在NOT IN子查询上。 而NOT IN子查询，可以等价改写成left join，改写形式如下：

```
P.ROLEID NOT IN
         (SELECT ROLEID
            FROM PER_LTE_ZIB_PB_COMMISSION_06 P
           WHERE P.OURFLAG || NVL(P.HOSTFLAG, 0) = '$$1000090000011')

改写成：

LEFT JOIN (SELECT ROLEID
            FROM PER_LTE_ZIB_PB_COMMISSION_06 P
           WHERE P.OURFLAG || NVL(P.HOSTFLAG, 0) = '$$1000090000011') D
        ON D.ROLEID=P.ROLEID  
 where  D.ROLEID IS NULL    
```

[SQL中带有NOT IN 子查询改写](https://blog.csdn.net/wanbin6470398/article/details/78709187)

## 数据处理经常出现数据倾斜，导致负载不均衡的问题，需要做统计分析找到倾斜数据特征，定散列策略。

## 使用Parquet列式存储，减少IO，提高Spark SQL效率。
