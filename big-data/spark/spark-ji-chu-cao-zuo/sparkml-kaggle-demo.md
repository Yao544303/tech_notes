# SparkML Kaggle Demo

## 数据预处理 Data preparation

### 转化成parquet：读csv文件，转为parquet

```
def convert(filePrefix : String) = { 
	val basePath = "yourBasePath"
	var df = spark.read
		.option("header",true)
		.option("inferSchema", "true")
		.csv("basePath+filePrefix+".csv") 
	df = df.repartition(1)
	df.write.parquet(basePath+filePrefix+".parquet")
}

convert("train_numeric") 
convert("train_date") 
convert("train_categorical")
```

### 读取这些文件为DataFrames

```
var df_numeric = spark.read.parquet(basePath+"train_numeric.parquet")
var df_categorical = spark.read.parquet(basePath+"train_categorical.parquet")
```

这是一个非常高维的数据，会使用一个子集

```
df_categorical.createOrReplaceTempView("dfcat")
var dfcat = spark.sql("select Id, L0_S22_F545 from dfcat")

df_numeric.createOrReplaceTempView("dfnum")
var dfnum = spark.sql("select Id,L0_S0_F0,L0_S0_F2,L0_S0_F4,Response from dfnum")
```

### 把这两部分join起来

```
var df = dfcat.join(dfnum,"Id") 
df.createOrReplaceTempView("df")
```

处理 NA :

```
var df_notnull = spark.sql(""" select Response as label, case when L0_S22_F545 is null then 'NA' else L0_S22_F545 end as L0_S22_F545,
case when L0_S0_F0 is null then 0.0 else L0_S0_F0 end as L0_S0_F0,  case when L0_S0_F2 is null then 0.0 else L0_S0_F2 end as L0_S0_F2,
case when L0_S0_F4 is null then 0.0 else L0_S0_F4 end as L0_S0_F4 from df """)
```

## 特征工程

```
import org.apache.spark.ml.feature.{OneHotEncoder, StringIndexer}
var indexer = new StringIndexer()
	.setHandleInvalid("skip")
	.setInputCol("L0_S22_F545")
	.setOutputCol("L0_S22_F545Index")
```

使用onehot

```
var indexed = indexer.fit(df_notnull).transform(df_notnull) indexed.printSchema

var encoder = new OneHotEncoder()
	.setInputCol("L0_S22_F545Index")
	.setOutputCol("L0_S22_F545Vec")

var encoded = encoder.transform(indexed)
```

整合特征 VectorAssembler

```
import org.apache.spark.ml.feature.VectorAssembler 
import org.apache.spark.ml.linalg.Vectors
var vectorAssembler = new VectorAssembler()
	.setInputCols(Array("L0_S22_F545Vec", "L0_S0_F0", "L0_S0_F2","L0_S0_F4"))
	.setOutputCol("features")
var assembled = vectorAssembler.transform(encoded)
```

产生的是一个系数矩阵

测试pipeline

```
import org.apache.spark.ml.Pipeline 
import org.apache.spark.ml.PipelineModel
//Create an array out of individual pipeline stages 
var transformers = Array(indexer,encoder,assembled)
var pipeline = new Pipeline().setStages(transformers).fit(df_notnull) 
var transformed = pipeline.transform(df_notnull)
```

### 训练模型

```
import org.apache.spark.ml.classification.RandomForestClassifier 
var rf = new RandomForestClassifier()
	.setLabelCol("label")
	.setFeaturesCol("features")

var model = new Pipeline().setStages(transformers :+ rf).fit(df_notnull) 
var result = model.transform(df_notnull)
```

### 模型调优

```
import org.apache.spark.ml.evaluation.BinaryClassificationEvaluator 
val evaluator = new BinaryClassificationEvaluator()
import org.apache.spark.ml.param.ParamMap
var evaluatorParamMap = ParamMap(evaluator.metricName -> "areaUnderROC") 
var aucTraining = evaluator.evaluate(result, evaluatorParamMap)
```

### CV

```
import org.apache.spark.ml.tuning.{CrossValidator, ParamGridBuilder} 
var paramGrid = new ParamGridBuilder()
	.addGrid(rf.numTrees, 3 :: 5 :: 10 :: 30 :: 50 :: 70 :: 100 :: 150 :: Nil)
	.addGrid(rf.featureSubsetStrategy, "auto" :: "all" :: "sqrt" :: "log2" :: "onethird" :: Nil)
	.addGrid(rf.impurity, "gini" :: "entropy" :: Nil)
	.addGrid(rf.maxBins, 2 :: 5 :: 10 :: 15 :: 20 :: 25 :: 30 :: Nil)
	.addGrid(rf.maxDepth, 3 :: 5 :: 10 :: 15 :: 20 :: 25 :: 30 :: Nil)
	.build()

var crossValidator = new CrossValidator()
	.setEstimator(new Pipeline().setStages(transformers :+ rf))
	.setEstimatorParamMaps(paramGrid)
	.setNumFolds(5)
	.setEvaluator(evaluator)

var crossValidatorModel = crossValidator.fit(df_notnull)

var newPredictions = crossValidatorModel.transform(df_notnull)
```

### 显示模型细节

```
var bestPipelineModel = crossValidatorModel.bestModel.asInstanceOf[PipelineModel]
var stages = bestPipelineModel.stages
import org.apache.spark.ml.classification.RandomForestClassificationModel 
val rfStage = stages(stages.length-1).asInstanceOf[RandomForestClassificationModel] 
rfStage.getNumTrees
rfStage.getFeatureSubsetStrategy 
rfStage.getImpurity 
rfStage.getMaxBins 
rfStage.getMaxDepth
```
