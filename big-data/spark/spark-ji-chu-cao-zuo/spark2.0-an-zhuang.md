# Spark2.0安装

## Spark 安装指南

```
该文档是建立在已经安装好Hadoop 和jdk基础上，并且已经设置好HADOOP_HOME环境变量及JAVA_HOME环境变量，测试和现网环境需要在原来的hadoop环境中安装。
```

## 1 安装配置Spark

### 1.1 下载安装

```
从Spark官网获取和hadoop版本相对应的Spark安装包。上传到服务器，解压。
```

```
	$ tar -zxvf spark-2.0.0-bin-hadoop2.7.tgz
```

```
配置环境变量 /etc/profile ,添加如下两行
```

```
	$ export SPARK_HOME = /housr/local/hadoop/spark-2.0.0-bin-hadoop2.7
	$ export PATH = $SPARK_HOME/bin:$PATH
```

### 1.2 配置

```
修改配置文件 Spark-env.sh（使用yarn 的话，不需要）
在主节点上进入Spark安装目录/conf 执行如下命令：
```

```
	$ cp spark-env.sh.template spark-env.sh
	$ vi spark-env.sh
	$ 添加hadoop 、scala、 java环境变量
```

### 1.3 启动

```
	$ cd/usr/local/spark
	$ ./bin/run-example SparkPi
```

```
或者，通过spark-submit 运行
```

```
	$ ./bin/spark-submit examples/src/main/python/pi.py
```

## 2 SparkSession 详解

```
Spark2.0中引入了SparkSession的概念，它为用户提供了一个统一的切入点来使用Spark的各项功能，用户不但可以使用DataFrame和Dataset的各种API，学习Spark的难度也会大大降低。
```

## 3 spark-submit

### Run application locally on 8 cores

```
./bin/spark-submit \
  --class org.apache.spark.examples.SparkPi \
  --master local[8] \
  /path/to/examples.jar \
  100
```

### Run on a Spark standalone cluster

```
./bin/spark-submit \
  --class org.apache.spark.examples.SparkPi \
  --master spark://207.184.161.138:7077 \
  --executor-memory 20G \
  --total-executor-cores 100 \
  /path/to/examples.jar \
  1000
```

### Run on a YARN cluster

```
export HADOOP_CONF_DIR=XXX
./bin/spark-submit \
  --class org.apache.spark.examples.SparkPi \
  --master yarn-cluster \  # can also be `yarn-client` for client mode
  --executor-memory 20G \
  --num-executors 50 \
  /path/to/examples.jar \
  1000
```

### Run a Python application on a cluster

```
./bin/spark-submit \
  --master spark://207.184.161.138:7077 \
  examples/src/main/python/pi.py \
  1000
```

## 4 Spark 数据挖掘

1. 使用Hive获取数据
2. 对获取的数据进行包装，适合机器学习算法
3. 使用机器学习的方法进行计算

## Spark 参数调优

最好用spark ui看慢在哪里了，然后再做优化。比如加载hdfs慢了，可能就和hadoop版本有关。
