# DT提供的代码中遇到的坑

## DT代码中的坑

连续两周时间都在支持DT以及相关的label的开发 能够明显的发现DT提供的代码质量非常之差。列举出来，前事不忘后事之师。

## hard core

在spark的代码中，将master 以及入参全部hard core，入参不必说他，将master 设置之后，我spark submit可是会报错的啊。

## 不转为String 直接saveAsTextFile

常常出现

```
(ABACD,ADF,1)

[Ljava.long.String;@76abcd405]
```

前者是元组直接输出，后者输出的是地址，写代码的时候一定需要注意

## 集群600G 内存，输入200G 全部cache

恩 cache 的确可以提高效率，但是你这个样子做，确定不会oom？

## 多次join

输入为(A,B,C) ,希望得到的输出(A,B/sum(B),B,D)，做了多次join，开销非常之大。\
在实践之前，可以先进行采样，加入对A进行reduce 之后 ，按key分布的数据量不大，倾斜不严重的情况下， 可以将join 转变 map 处理为 RDD\[String,Map]，再reduce， 得到RDD\[String,Map]之后，在map内部进行相似逻辑的操作，这样能提高效率。

## 不做异常检测

维表可能存在空值，不做异常检测，直接进行string => int 的转化，必然异常。

## 过滤数据

接上，对空值的过滤，需要谨慎再谨慎，每条数据都是很宝贵的，需要非常认真的对待，建议在filter之前，先sample一下，看看数据是什么样子，看看要filter的数据是什么样子，再做决断。

## sample的重要性

既然用到了spark 处理的数据量级自然不会小，在大数据量测试之前务必使用小数据量进行逻辑的验证，直接用大数据量跑的话，耗时耗资源不去说，万一错了，代价也很大。

## 维表过多

维表过多，导致管理起来非常困难，一定要协商好一个更新机制

## Spark-submit 脚本

这个必须有，整理的晚了，每次提交都要重新编写，虽然时间不多，但多几次，很容易让人狂躁 整理了如下一个模板 [spark-submit模板](https://github.com/Yao544303/shell/blob/master/java/spark\_submit.sh)

## 信息沟通必须及时
