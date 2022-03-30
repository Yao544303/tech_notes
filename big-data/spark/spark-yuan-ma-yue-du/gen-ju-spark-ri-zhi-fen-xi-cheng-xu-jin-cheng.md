# 根据Spark 日志，分析程序进程

根据Spark日志 学习Spark 处理流程

## 背景

使用的是spark on Yarn 模式，提交参数如下： spark-submit --master yarn-client --executor-memory 6g --conf spark.executor.instances=20 --class ${MAIN} ${JAR} ${args} > logstats 2>&1 &

## 日志分析

### 第一部分

```
18/04/04 15:01:27 INFO spark.SparkContext: Running Spark version 1.6.0
```

* 1 启动Spark，

```
18/04/04 15:01:28 WARN spark.SparkConf:
SPARK_CLASSPATH was detected (set to ':/opt/cloudera/parcels/HADOOP_LZO/lib/hadoop/lib/*').
This is deprecated in Spark 1.0+.
Please instead use:
 - ./spark-submit with --driver-class-path to augment the driver classpath
 - spark.executor.extraClassPath to augment the executor classpath

18/04/04 15:01:28 WARN spark.SparkConf: Setting 'spark.executor.extraClassPath' to ':/opt/cloudera/parcels/HADOOP_LZO/lib/hadoop/lib/*' as a work-around.
18/04/04 15:01:28 WARN spark.SparkConf: Setting 'spark.driver.extraClassPath' to ':/opt/cloudera/parcels/HADOOP_LZO/lib/hadoop/lib/*' as a work-around.
```

* 2 以上部分在SparkConf 的validateSettings() 中，校验到SPARK\_CLASSPATH 这个环境变量设置了，将driver 和 executor 的相关设置都改为这个。

```
18/04/04 15:01:28 INFO spark.SecurityManager: Changing view acls to: miao18
18/04/04 15:01:28 INFO spark.SecurityManager: Changing modify acls to: miao18
18/04/04 15:01:28 INFO spark.SecurityManager: SecurityManager: authentication disabled; ui acls disabled; users with view permissions: Set(miao18); users with modify permissions: Set(miao18)
```

* 3 以上部分是SparkContext 初始化，调用org.apache.spark.SparkEnv.createDriverEnv -> create -> 初始化 val securityManager = new SecurityManager(conf) -> setViewAcls setModifyAcls logInfo

```
18/04/04 15:01:28 INFO util.Utils: Successfully started service 'sparkDriver' on port 59965.
18/04/04 15:01:28 INFO slf4j.Slf4jLogger: Slf4jLogger started
18/04/04 15:01:28 INFO Remoting: Starting remoting
18/04/04 15:01:28 INFO Remoting: Remoting started; listening on addresses :[akka.tcp://sparkDriverActorSystem@192.168.111.127:40690]
18/04/04 15:01:28 INFO Remoting: Remoting now listens on addresses: [akka.tcp://sparkDriverActorSystem@192.168.111.127:40690]
18/04/04 15:01:28 INFO util.Utils: Successfully started service 'sparkDriverActorSystem' on port 40690.
```

* 4 启动远程监听服务，Spark的通信工作由akka来实现

```
18/04/04 15:01:28 INFO spark.SparkEnv: Registering MapOutputTracker
18/04/04 15:01:28 INFO spark.SparkEnv: Registering BlockManagerMaster
18/04/04 15:01:28 INFO storage.DiskBlockManager: Created local directory at /tmp/blockmgr-183c7bab-bc15-45a7-8cbc-e1f09f2421b2
18/04/04 15:01:28 INFO storage.MemoryStore: MemoryStore started with capacity 530.0 MB
18/04/04 15:01:28 INFO spark.SparkEnv: Registering OutputCommitCoordinator
18/04/04 15:01:28 WARN util.Utils: Service 'SparkUI' could not bind on port 4040. Attempting port 4041.
18/04/04 15:01:28 INFO util.Utils: Successfully started service 'SparkUI' on port 4041.
18/04/04 15:01:28 INFO ui.SparkUI: Started SparkUI at http://192.168.111.127:4041
18/04/04 15:01:28 INFO spark.SparkContext: Added JAR file:/data/users/yaochengzong/dt/test/dt_all-1.0-SNAPSHOT.jar at spark://192.168.111.127:59965/jars/dt_all-1.0-SNAPSHOT.jar with time
stamp 1522825288973
18/04/04 15:01:29 INFO yarn.Client: Requesting a new application from cluster with 11 NodeManagers
18/04/04 15:01:29 INFO yarn.Client: Verifying our application has not requested more than the maximum memory capability of the cluster (8192 MB per container)
18/04/04 15:01:29 INFO yarn.Client: Will allocate AM container, with 896 MB memory including 384 MB overhead
18/04/04 15:01:29 INFO yarn.Client: Setting up container launch context for our AM
18/04/04 15:01:29 INFO yarn.Client: Setting up the launch environment for our AM container
18/04/04 15:01:29 INFO yarn.Client: Preparing resources for our AM container
18/04/04 15:01:30 INFO yarn.Client: Uploading resource file:/tmp/spark-7aead3cb-ea39-474a-a707-1b6c3ff0e16b/__spark_conf__4513331269041362768.zip -> hdfs://nameservice/user/yaochengzong/
.sparkStaging/application_1522199761158_12819/__spark_conf__4513331269041362768.zip
18/04/04 15:01:30 INFO spark.SecurityManager: Changing view acls to: yaochengzong
18/04/04 15:01:30 INFO spark.SecurityManager: Changing modify acls to: yaochengzong
18/04/04 15:01:30 INFO spark.SecurityManager: SecurityManager: authentication disabled; ui acls disabled; users with view permissions: Set(yaochengzong); users with modify permissions: S
et(yaochengzong)
18/04/04 15:01:30 INFO yarn.Client: Submitting application 12819 to ResourceManager
18/04/04 15:01:30 INFO impl.YarnClientImpl: Submitted application application_1522199761158_12819
```

* 5 调用链 org.apache.spark.SparkEnv.createDriverEnv -> create -> ActorSystem -> 根据条件，AkkaRpcEnv 中调用AkkaUtils 或者 直接调用 AkkaUtils -> createActorSystem -> Utils.startServiceOnPort
