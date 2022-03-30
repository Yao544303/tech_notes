# R\_Tips

## 读取|分割文件

```
test<-read.csv("C:/Users/Administrator/Desktop/tmp/part-00000",sep='|',head =
FALSE,fileEncoding="UTF-8")
if (nchar(Sys.getenv("SPARK_HOME")) < 1) {
Sys.setenv(SPARK_HOME = "/home/spark")
}
library (SparkR, lib.loc = c(file.path(Sys.getenv("SPARK_HOME"), "R", "lib")))
sc <- sparkR.init(master = "local[*]", sparkEnvir = list(spark.driver.memory="2g"))
```

## 设置工作区

```
setwd('C:/workspace/workspace4R/xd')
```

## 加载

```
load(‘colation.RData’)
```

## R使用pmml

```
library("randomForest")
rf = randomForest(Species ~ ., data = iris)
saveRDS(rf, "rf.rds")
java -jar target/converter-executable-1.3-SNAPSHOT.jar --rds-input rf.rds --pmml-
output rf.pmml
```
