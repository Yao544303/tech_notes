# SparkML 采坑记录

## parquet to dataframe

* 字段名称一定要对上
* 字段类型一定要对上，例如 double 和 float 要对上

## Spark 1.6.3 ML 中的RF 无法存储

固有bug，使用如下方式补救： 存储

```
val rf = new RandomForestClassifier()
    .setLabelCol("indexedLabel")
    .setFeaturesCol("scaledFeatures")
    .setNumTrees(modelProperties.getOrElse("randomforest.numtrees", "700").toInt)
    .setMaxDepth(modelProperties.getOrElse("randomforest.maxdepth", "30").toInt)

println("finish trained")
val model = rf.fit(trainingData)
sc.parallelize(Seq(model),1).saveAsObjectFile(modelSave)
```

加载：

```
val rf = sc.objectFile[RandomForestClassificationModel](modelPath).first()
```

## ml Vs mllib

* ml 的话，把所有features 组织成向量的形式
* mllib 的话，把所有features 组织成labelPoint 的形式 ml的本质还是调用mllib
* 社区一直在说，要废弃RDD，只用dataset 和 dataframe 比如现在新出的structStream 就把Streaming 变成一张很宽的表，然后 ml 也不再调用mllib。但目前还是这样

## stringIndex

按String字符串出现次数降序排序；次数相同，自然顺序。按降序顺序转换成DoubleIndex，默认从0.0开始。

需要介绍一下这里，为什么要to Index 你输入的分类 有AB,有男女，有高矮胖瘦，这个，对模型来说，很麻烦，所以干脆统一编号，0 1 2 3 4，判断出来了再映射一次

在应用StringIndexer对labels进行重新编号后，带着这些编号后的label对数据进行了训练，并接着对其他数据进行了预测，得到预测结果，预测结果的label也是重新编号过的，因此需要转换回来。

## VectorIndexer

解决数据集中的类别特征Vector。它可以自动识别哪些特征是类别型的，并且将原始值转换为类别指标。它的处理流程如下：

```
Step1.获得一个向量类型的输入以及maxCategories参数。
Step2.基于原始数值识别哪些特征需要被类别化，其中最多maxCategories需要被类别化。
Step3.对于每一个类别特征计算0-based类别指标。
Step4.对类别特征进行索引然后将原始值转换为指标。
```

## 构建pipeline 时候注意schema 的名字

例子

```
训练数据有 label features 两列
预测数据有 userid features 两列
```

经过的转化为

```
val minMaxScaler = new MinMaxScaler()
    .setInputCol("features")
    .setOutputCol("scaledFeatures")
val labelIndexer = new StringIndexer()
    .setInputCol("label")
    .setOutputCol("indexedLabel")
val rf = new RandomForestClassifier()
    .setLabelCol("indexedLabel")
    .setFeaturesCol("scaledFeatures")    
```

如果如下构建pipeline

```
val pipeline = new Pipeline().setStages(Array(minMaxScaler, labelIndexer, rf))
```

用来预测的时候会报错，因为预测数据没有label

## 版本差异，导致API差异，尽量用scala 开发吧

* pyspark 1.6.3 中没有 ml rf 的API，没有LDA 的API
* pyspark 部分mllib 有的api ，ml 中却没有

## 标签和索引的转化： StringInderxer- IndexToString - VectorIndexer

Spark的机器学习处理过程中，经常需要把标签数据（一般是字符串）转化成整数索引，而在计算结束又需要把整数索引还原为标签。这就涉及到几个转换器：StringIndexer、 IndexToString，OneHotEncoder，以及针对类别特征的索引VectorIndexer。

## is apache spark less accurate than scikit learn

SGD(梯度下降)是一种在线的凸优化方法，因此，很难并行化。 相对的，L-BFGS，就很容易并行化，所以LogigisticRegressionWithLBFGS 更好用点 [ref](https://stackoverflow.com/questions/28076232/is-apache-spark-less-accurate-than-scikit-learn)
