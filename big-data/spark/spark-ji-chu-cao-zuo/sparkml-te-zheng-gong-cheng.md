# SparkML 特征工程

## Summary statistics

统计特征，包括：max, min, mean, variance, and number of nonzeros（非0值个数）, as well as the total count. variance方差，normL1:L1范数 绝对值相加，normL2:L2范数 平方和

```
val dataset = sqlContext.sql("SELECT * FROM parquet.`%s`".replaceAll("%s", parquet_data_path_training)).toDF("features", "label")
val a = dataset.rdd.map(x=>x(0).asInstanceOf[org.apache.spark.mllib.linalg.Vector])
val stat1 = Statistics.colStats(a)
stat1.numNonzeros
```

## Correlations

相关性分析

```
val correlMatrix  = Statistics.corr(a, "pearson")
```

### 两种相关系数

* pearson：衡量两个数据集的 线性相关 程度
* spearman： 相关系数不关心两个数据集是否线性相关，而是单调相关

## Stratified sampling

分层采样

## Hypothesis testing

假设检验 卡方检验

```
val vec: Vector = ... // a vector composed of the frequencies of events

// compute the goodness of fit. If a second vector to test against is not supplied as a parameter, 
// the test runs against a uniform distribution.  
val goodnessOfFitTestResult = Statistics.chiSqTest(vec)
println(goodnessOfFitTestResult) // summary of the test including the p-value, degrees of freedom, 
                                 // test statistic, the method used, and the null hypothesis.

val mat: Matrix = ... // a contingency matrix

// conduct Pearson's independence test on the input contingency matrix
val independenceTestResult = Statistics.chiSqTest(mat) 
println(independenceTestResult) // summary of the test including the p-value, degrees of freedom...

val obs: RDD[LabeledPoint] = ... // (feature, label) pairs.

// The contingency table is constructed from the raw (feature, label) pairs and used to conduct
// the independence test. Returns an array containing the ChiSquaredTestResult for every feature 
// against the label.
val featureTestResults: Array[ChiSqTestResult] = Statistics.chiSqTest(obs)
var i = 1
featureTestResults.foreach { result =>
    println(s"Column $i:\n$result")
    i += 1
} // summary of the test
```

## Random data generation

随机生成数据

```
val sc: SparkContext = ...

// Generate a random double RDD that contains 1 million i.i.d. values drawn from the
// standard normal distribution `N(0, 1)`, evenly distributed in 10 partitions.
val u = normalRDD(sc, 1000000L, 10)
// Apply a transform to get a random double RDD following `N(1, 4)`.
val v = u.map(x => 1.0 + 2.0 * x)
```

## Kernel density estimation

核密度估计

```
val data: RDD[Double] = ... // an RDD of sample data

// Construct the density estimator with the sample data and a standard deviation for the Gaussian
// kernels
val kd = new KernelDensity()
  .setSample(data)
  .setBandwidth(3.0)

// Find density estimates for the given values
val densities = kd.estimate(Array(-1.0, 2.0, 5.0))
```

## ref

* [Spark MLlib学习——特征工程](https://www.jianshu.com/p/e662daa8970a)
* [spark机器学习库评估指标总结](https://blog.csdn.net/u011707542/article/details/77838588)
