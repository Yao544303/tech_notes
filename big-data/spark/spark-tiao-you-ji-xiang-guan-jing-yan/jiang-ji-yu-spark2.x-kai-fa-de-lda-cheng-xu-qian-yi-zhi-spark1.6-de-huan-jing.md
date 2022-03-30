# 将基于Spark2.x开发的LDA程序迁移至Spark1.6的环境

## 前言

对方提供的LDA 聚类程序，是基于Spark 2.x 的，但是，我们的生产环境是Spark 1.6。 恩那么问题就来了，怎么让基于Spark 2.x 的代码在Spark 1.6 上跑起来。第一个想法是，只要把SparkSession 改为SparkContext 和 SQLContext 就行了。然后发现自己真的图样图森破。对方非常高端的使用了ml库，当然这不是问题。对方是用python 写的，当然这也不是问题~~毕竟小学生都要学python了，作为一个程序员不会python 就说不过去了~~对方的python 版本是2.7，当然这更不是问题。但是当你遇到需要将python 2.7 写的基于Spark 2.x ml 写的代码迁移到spark 1.6 上的时候，就有问题了。 ~~他喵的，老子都要精分了，一直在python 2.7 python 3.0 spark 1.6 spark2.2 版本间切换，在java，scala ，python 语言间切换~~

## 改造经过

任务到手，谋定后动，当然先开始分析。

```
	spark = SparkSession.builder.master('yarn') \
		.appName('marketing_data_lda_clustering') \
		.enableHiveSupport() \
		.getOrCreate()
```

这个，当然小case啦。改成sparkContext 就行了

```
	conf = SparkConf().setMaster('yarn') \
		.setAppName('marketing_data_lda_clustering') 
	sc = SparkContext(conf = conf)	
	sqlContext = SQLContext(sc)
```

然后，以为就搞定了，开开心心的spark-submit。不出意外的报错，提示下面代码缺少方法。

```
	lda = LDA(k=30, seed=long(time.time()), optimizer="em")
	model = lda.fit(feature_dataframe)
```

怎么可能！！！ ml 怎么可能没有fit？？？ 带着满腔疑惑，打开了api，然后发现，不是ml 没有fit，是1.6 的ml 库中压根就没有LDA！！！ 哦对，顺便看了眼隔壁Scala 的API 两个都有。。。 ~~心情复杂~~ 看来只能将ml 的程序改成mllib了~~毕竟我当年也改过，只不过是java 的而已~~ 说干就干，首先将fit 改为train，然后将输入从dataframe 改为RDD 即可。因为构建features的时候，对方已经将df 转为了rdd，只需要将对方从rdd转为df 的那一步去掉就行了。~~简直完美~~

```
features = data.rdd.map(lambda x: (x[0], x[1:])).groupByKey().mapValues(list).map(lambda row: build_sparse_vector(row, tag_index_map)).cache()
model = LDA.train(rdd, k=10, seed=1)
```

然后，不出意外的报错了，类型不匹配。 怎么会类型不匹配呢？一定是我对LDA 的算法理解不够深入。干脆找个简单的例子来测试下吧。 于是照着example 撸了下面的代码

```
conf = SparkConf().setMaster('local').setAppName('marketing_data_lda_clustering') 
sc = SparkContext(conf = conf)	
sqlContext = SQLContext(sc)
data = [
     [1, Vectors.dense([0.0, 1.0])],
     [2, SparseVector(2, {0: 1.0})],
]
rdd =  sc.parallelize(data)
model = LDA.train(rdd, k=2, seed=1)
print(model.vocabSize())

print(model.describeTopics())
print(model.describeTopics(1))
```

结果一次通过，正确无比。那么问题究竟出在哪里呢？ 算法的各项参数差异不大，应该不会出问题。有问题的就只能是rdd了。python 新学，不会debug，只能祭出print 大法了~~顺便吐槽下，能让服务器版本2.7 driver python 版本2.6 的 也是醉了~~ 将测试用的rdd collect 然后打印出来的结果如下：

```
[[1, DenseVector([0.0, 1.0])], [2, SparseVector(2, {0: 1.0})]]	
```

有问题的代码打出来是

```
[(u'D5A55D4E4E8F484FB4ACFCCAE1E37BFE', SparseVector(10, {1: 1.0, 7: 1.0})), (u'41AF98FE050658F50F0DD55F823CE954', SparseVector(10, {2: 1.0, 4: 1.0})), (u'4CD09883D04D35697666115ADE9393DC', SparseVector(10, {5: 1.0, 6: 1.0, 8: 1.0, 9: 1.0})), (u'CF5F565193E6FE9B5D7D50F3DADCB85F', SparseVector(10, {3: 1.0})), (u'E776CC8BAF4900C092D9789077EF1E56', SparseVector(10, {0: 1.0}))]	
```

这他妈坑爹呢！！！ 对方的构造方法返回居然是这个格式，无奈在features 加上如下的map 方法

```
.map(lambda x: [x[0], x[1]])
```

然而报错依赖。绝望之际，只能去读源码了。以前一直读的是scala，不知道python的源码会怎们样？ 怀着激动的心情，我打开了spark源码。 机智如我，一下子就找到了python\pyspark\mllib 路径下的clustering.py文件~~这不是废话么~~ 找到了LDA的实现 关键的实现，就这么一句

```
        model = callMLlibFunc("trainLDAModel", rdd, k, maxIterations,
                              docConcentration, topicConcentration, seed,
                              checkpointInterval, optimizer)
```

这个callMLlibFunc 方法好厉害，看了好几个方法都是调用这个。于是就追到这个方法里看了下。 结果

```
def callJavaFunc(sc, func, *args):
    """ Call Java Function """
    args = [_py2java(sc, a) for a in args]
    return _java2py(sc, func(*args))


def callMLlibFunc(name, *args):
    """ Call API in PythonMLLibAPI """
    sc = SparkContext.getOrCreate()
    api = getattr(sc._jvm.PythonMLLibAPI(), name)
    return callJavaFunc(sc, api, *args)
```

他喵的，原来pyspark mllib 的底层实现是调了jar 。真是简洁的设计呢。

原来python并不是实现。。正当绝望之时，想起当时调试LR，被labeledpoint 支配的恐惧。难道id 输入不能是string？和graphx 一样都必须转化为long？python既然依赖于scala 的源码，那查scala 的源码岂不是一样？ 于是查啊查，查到LDA 的scala 执行 在org.apache.spark.mllib.clustering.LDA.scala 中查到有个run

```
  def run(documents: JavaPairRDD[java.lang.Long, Vector]): LDAModel = {
    run(documents.rdd.asInstanceOf[RDD[(Long, Vector)]])
  }
```

好了很明确了，就是要转成long，那df 为什么不用转呢？

又开始查ml的源码

在fit 方法中，有这么一段

```
val oldData = LDA.getOldDataset(dataset, $(featuresCol))
```

我能吐槽么。。。 老版本的能用的，直接加个前缀。。。。。。 然后再跟到vgetOldDataset

```
  /** Get dataset for spark.mllib LDA */
  private[clustering] def getOldDataset(
       dataset: Dataset[_],
       featuresCol: String): RDD[(Long, OldVector)] = {
    dataset
      .withColumn("docId", monotonically_increasing_id())
      .select("docId", featuresCol)
      .rdd
      .map { case Row(docId: Long, features: Vector) =>
        (docId, OldVectors.fromML(features))
      }
  }
```

连带着注释一起贴了，说好的ml 是对df 有了优化的呢，他喵的，这不是 还是转成mllib 实现了么！！！ 所以，问题其实已经解决了，就是这一句

```
withColumn("docId", monotonically_increasing_id())
```

生成了一个long 类型的id。 ok 完美落幕，再一次愉快的submit。 然后 WTF！！！！！ 又他娘的报错。 从describeTopics的结果生成df 的时候发错，然后发现。 RDD describeTopics的结果是这样的

```
[([7, 8, 1, 0, 4, 3, 2, 6, 9, 5], [0.10368743342260674, 0.10207234154290727, 0.10138905514695376, 0.10061123338885637, 0.09970383057872025, 0.09948034467474759, 0.09945100732346705, 0.09850959571098065, 0.09837913555700037, 0.09671602265375995]), ([5, 9, 6, 2, 3, 4, 0, 1, 8, 7], [0.10328397706229461, 0.10162086430285339, 0.1014904041601532, 0.10054899262906491, 0.100519655280321, 0.10029616939567182, 0.09938876666399324, 0.09861094497314936, 0.09792765863627545, 0.09631256689622307])]
```

DF describeTopics 的结果在前面有个编号。是ML 在调用时有入参，做了一次累加。

```
def describeTopics(maxTermsPerTopic: Int)
```

so 略作调整，通过row number 加了个字段搞定。至此 告一段路
