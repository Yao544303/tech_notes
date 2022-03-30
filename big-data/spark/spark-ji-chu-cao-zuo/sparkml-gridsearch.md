# SparkML  GridSearch

## Spark ML Grid Search

```
val kmeans = new KMeans().setK(2).setMaxIter(20).setFeaturesCol("features").setPredictionCol("prediction")  
//主要问题在这里，没有可用的评估器与label列设置  
val evaluator = new BinaryClassificationEvaluator().setLabelCol("prediction")  
val paramGrid = new ParamGridBuilder().addGrid(kmeans.initMode, Array("random")).addGrid(kmeans.k, Array(3, 4)).addGrid(kmeans.maxIter, Array(20, 60)).addGrid(kmeans.seed, Array(1L, 2L)).build()  
val steps: Array[PipelineStage] = Array(kmeans)  
val pipeline = new Pipeline().setStages(steps)  
  
val cv = new CrossValidator().setEstimator(pipeline).setEvaluator(evaluator).setEstimatorParamMaps(paramGrid).setNumFolds(10)  
// Trains a model  
val pipelineFittedModel = cv.fit(dataset)
```
