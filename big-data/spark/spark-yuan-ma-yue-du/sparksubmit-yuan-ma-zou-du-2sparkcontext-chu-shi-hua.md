# SparkSubmit源码走读2\_SparkContext初始化

上文，runMain 设置MainClass 参数，部署方式等，最后通过mainMethod.invoke(null, childArgs.toArray) 通过反射机制，找到需要执行的类，开始执行。

## 第一步，初始化SparkConf， SparkConf用于维护Spark的配置属性。

### SparkConf 定义了2个变量

* setting:ConcurrentHashMapString, String 用来保存各种设置
* avroNamespace 定义avro的schema空间

### SparkConf 中的方法主要有如下几类

* set类方法，为sparkConf 中setting添加k-v配置项
* get类方法，获取sparkConf 中的配置项
* get单位类方法
* 校验类主要校验参数是否过时，参数是否符合要求
* 其他基本操作

构造方法会将systemProperties 中所有以"spark." 开头的配置项set入setting。一般在代码中是如下格式

```
    val sparkConf = new SparkConf().setAppName("dt hour").setMaster("local")
    val sc = new SparkContext(sparkConf)
```

也就是在sparkConf 初始化之后，再调用set方法，故而，在代码中的set参数 优先级最高 由于SparkConf提供的setter方法返回的是this，也就是一个SparkConf对象，所有它允许使用链式来设置属性。

## 第二步，SparkContext初始化

1. 主要流程

```
sparkContext 创建和初始化的变量（非标注，即是初始化），以及相关启动操作
-----> 复制SparkConf信息，并校验
-----> 根据SparkConf 设置一系列参数
-----> SparkEnv 
		* SparkEnv包含了Spark实例（master or worker）运行时的环境对象，包括serializer, Akka actor system, block manager, map output tracker等等
	-----> RPCEnv
		----->EndpointRef
-----> MetadataCleancer 
		* 在SparkContext初始化的过程中，会创建一个MetadataCleaner，用于清理缓存到内存中的RDD。MetadataCleaner是用来定时的清理metadata的，metadata有6种类型，封装在了MetadataCleanerType类中。
		** MAP_OUTPUT_TRACKER：map任务的输出元数据
		** SPARK_CONTEXT：缓存到内存中的RDD
		** HTTP_BROADCAST：采用http方式广播broadcast的元数据
		** BLOCK_MANAGER：BlockManager中非Broadcast类型的Block数据
		** SHUFFLE_BLOCK_MANAGER：shuffle输出的数据
		** BROADCAST_VARS：Torrent方式广播broadcast的元数据，底层依赖于BlockManager。
-----> SparkStatusTracker
		* SparkStatusTracker是低级别的状态报告API，用于监控job和stage。
-----> ConsoleProgressBar
		* 进度条
-----> SparkUI
		* SparkUI为Spark监控Web平台提供了Spark环境、任务的整个生命周期的监控。
-----> HadoopConfiguration
		* 由于Spark默认使用HDFS作为分布式文件系统，所以需要获取Hadoop相关的配置信息。
		** 将Amazon S3文件系统的AccessKeyId和SecretAccessKey加载到Hadoop的Configuration。
		** 将SparkConf中所有以spark.hadoop.开头的属性复制到Hadoop的Configuration。
		** 将SparkConf的spark.buffer.size属性复制为Hadoop的Configuration的io.file.buffer.size属性。
-----> ExecutorEnvs		
		** ExecutorEnvs包含的环境变量会在注册应用时发送给Master，Master给Worker发送调度后，Worker最终使用executorEnvs提供的信息启动Executor。
-----> HeartbeatReciver
-----> TaskScheduler
	    * askScheduler为Spark的任务调度器，Spark通过它提交任务并且请求集群调度任务；TaskScheduler通过master的配置匹配部署模式，创建TashSchedulerImpl，根据不同的集群管理模式（local、local[n]、standalone、local-cluster、mesos、YARN）创建不同的SchedulerBackend。
	-----> TaskSchedulerImpl
	-----> Backend（local 或者其他）
	    * 构造过程
		** 从SparkConf 中读取配置信息，包括每个任务分配的CPU数，调度策略(FIFO,FAIR) 
		** 创建TaskResultGetter:通过线程池对Worker上的Executor发送的Task的执行结果进行处理。默认会通过Executors.newFixedThreadPool创建一个包含4个、线程名以task-result-getter开头的线程池。
		** TaskSchedulerImpl的初始化
		*** 使TaskSchedulerImpl持有LocalBackend的引用
		*** 创建Pool，Pool中缓存了调度队列，调度算法以及TaskSetManager集合等信息。
		*** 创建FIFOSchedulableBuilder，FIFOSchedulableBuilder用来操作Pool中的调度队列。	
-----> DAGScheduler
		*  DAGScheduler的主要作用是在TaskSchedulerImpl正式提交任务之前做一些准备工作，包括：创建Job，将DAG中的RDD划分到不同的Stage，提交Stage等等
-----> _heartbeatReceiver.ask[Boolean](TaskSchedulerIsSet) 发出心跳信息，验证TaskScheduluer是否设置好		
----->  _taskScheduler.start() 并绑定appid 等参数，初始化blockmanger
-----> metricsSystem.start
-----> EventLoggingListener 初始化
-----> ExecutorAllocationManger	初始化
-----> ContextCleaner 初始化
-----> setupAndStartListenerBus() 
-----> postEnvironmentUpdate()
-----> postApplicationStart()
-----> 初始化Post 
-----> ShutdownHookManager 用来保证sc stop了
```

1. 初始化的主要实现部分

```
// The call site where this SparkContext was constructed.
private val creationSite: CallSite = Utils.getCallSite()

// If true, log warnings instead of throwing exceptions when multiple SparkContexts are active
private val allowMultipleContexts: Boolean =
  config.getBoolean("spark.driver.allowMultipleContexts", false)

// In order to prevent multiple SparkContexts from being active at the same time, mark this
// context as having started construction.
// NOTE: this must be placed at the beginning of the SparkContext constructor.
SparkContext.markPartiallyConstructed(this, allowMultipleContexts)
```

getCallSite方法会得到一个CallSite对象，改对象存储了线程栈中最靠近栈顶的用户类及最靠近栈底的Scala或者Spark核心类信息。 SparkContext默认只有一个实例，由属性"spark.driver.allowMultipleContexts"来控制。 markPartiallyConstructed方法用于确保实例的唯一性，并将当前SparkContext标记为正在构建中。

```
// log out Spark Version in Spark driver log  
logInfo(s"Running Spark version $SPARK_VERSION")
```

查看spark的日志，第一句基本是这个

```
try {
    // 复制sparkConf，并校验
    _conf = config.clone()
    // sparkConf 校验，日志中的
    // SPARK_CLASSPATH was detected (set to ':/opt/cloudera/parcels/HADOOP_LZO/lib/hadoop/lib/*').This is deprecated in Spark 1.0+.
    // 就是因为这一步
    _conf.validateSettings()

    // 校验是否设置了master 和 appname，这两个不设置会抛出异常
    if (!_conf.contains("spark.master")) {
      throw new SparkException("A master URL must be set in your configuration")
    }
    if (!_conf.contains("spark.app.name")) {
      throw new SparkException("An application name must be set in your configuration")
    }



    // System property spark.yarn.app.id must be set if user code ran by AM on a YARN cluster
    // yarn-standalone is deprecated, but still supported
    // spark.yarn.app.id 这个参数在yarn-cluster yarn-standalone 模式下不能设置
    if ((master == "yarn-cluster" || master == "yarn-standalone") &&
        !_conf.contains("spark.yarn.app.id")) {
      throw new SparkException("Detected yarn-cluster mode, but isn't running on a cluster. " +
        "Deployment to YARN is not supported directly by SparkContext. Please use spark-submit.")
    }

    if (_conf.getBoolean("spark.logConf", false)) {
      logInfo("Spark configuration:\n" + _conf.toDebugString)
    }

    
    // Set Spark driver host and port system properties
    // 这个是这是deploymode 的，当为空时，默认driver 为localhost 端口为0
    _conf.setIfMissing("spark.driver.host", Utils.localHostName())
    _conf.setIfMissing("spark.driver.port", "0")

    _conf.set("spark.executor.id", SparkContext.DRIVER_IDENTIFIER)

    // 依赖jar包，命令行 --jars 参数
    _jars = _conf.getOption("spark.jars").map(_.split(",")).map(_.filter(_.size != 0)).toSeq.flatten
    // 配置文件， 命令行 --files 参数
    _files = _conf.getOption("spark.files").map(_.split(",")).map(_.filter(_.size != 0))
      .toSeq.flatten

    // eventlog 目录，先判断是否开启，默认是开启的  EventLoggingListener.DEFAULT_LOG_DIR 默认值为 /tmp/spark-events
    _eventLogDir =
      if (isEventLogEnabled) {
        val unresolvedDir = conf.get("spark.eventLog.dir", EventLoggingListener.DEFAULT_LOG_DIR)
          .stripSuffix("/")
        Some(Utils.resolveURI(unresolvedDir))
      } else {
        None
      }

    // eventLog 压缩格式,有 lzf，snappy 可选
    _eventLogCodec = {
      val compress = _conf.getBoolean("spark.eventLog.compress", false)
      if (compress && isEventLogEnabled) {
        Some(CompressionCodec.getCodecName(_conf)).map(CompressionCodec.getShortName)
      } else {
        None
      }
    }

    // block文件的外部管理器
    _conf.set("spark.externalBlockStore.folderName", externalBlockStoreFolderName)

    if (master == "yarn-client") System.setProperty("SPARK_YARN_MODE", "true")

    // "_jobProgressListener" should be set up before creating SparkEnv because when creating
    // "SparkEnv", some messages will be posted to "listenerBus" and we should not miss them.
    // 初始化一个监听器，调用其构造函数，只初始化一些变量
    _jobProgressListener = new JobProgressListener(_conf)
    listenerBus.addListener(jobProgressListener)

    // Create the Spark execution environment (cache, map output tracker, etc)
    // 下面重要的是sparkEnv的初始化，sparkEnv是spark的执行环境对象，包括很多与Executor执行相关的对象。具体见下一部分
    _env = createSparkEnv(_conf, isLocal, listenerBus)
    SparkEnv.set(_env)

    // 元数据清理    
    _metadataCleaner = new MetadataCleaner(MetadataCleanerType.SPARK_CONTEXT, this.cleanup, _conf)

    // 状态跟踪
    _statusTracker = new SparkStatusTracker(this)

    _progressBar =
      if (_conf.getBoolean("spark.ui.showConsoleProgress", true) && !log.isInfoEnabled) {
        Some(new ConsoleProgressBar(this))
      } else {
        None
      }

    _ui =
      if (conf.getBoolean("spark.ui.enabled", true)) {
        Some(SparkUI.createLiveUI(this, _conf, listenerBus, _jobProgressListener,
          _env.securityManager, appName, startTime = startTime))
      } else {
        // For tests, do not enable the UI
        None
      }
    // Bind the UI before starting the task scheduler to communicate
    // the bound port to the cluster manager properly
    _ui.foreach(_.bind())

    _hadoopConfiguration = SparkHadoopUtil.get.newConfiguration(_conf)

    // Add each JAR given through the constructor
    if (jars != null) {
      jars.foreach(addJar)
    }

    if (files != null) {
      files.foreach(addFile)
    }

    // 设置executormenory ，可以看到都不设置默认1024
    _executorMemory = _conf.getOption("spark.executor.memory")
      .orElse(Option(System.getenv("SPARK_EXECUTOR_MEMORY")))
      .orElse(Option(System.getenv("SPARK_MEM"))
      .map(warnSparkMem))
      .map(Utils.memoryStringToMb)
      .getOrElse(1024)

    // Convert java options to env vars as a work around
    // since we can't set env vars directly in sbt.
    for { (envKey, propKey) <- Seq(("SPARK_TESTING", "spark.testing"))
      value <- Option(System.getenv(envKey)).orElse(Option(System.getProperty(propKey)))} {
      executorEnvs(envKey) = value
    }
    Option(System.getenv("SPARK_PREPEND_CLASSES")).foreach { v =>
      executorEnvs("SPARK_PREPEND_CLASSES") = v
    }
    // The Mesos scheduler backend relies on this environment variable to set executor memory.
    // TODO: Set this only in the Mesos scheduler.
    executorEnvs("SPARK_EXECUTOR_MEMORY") = executorMemory + "m"
    executorEnvs ++= _conf.getExecutorEnv
    executorEnvs("SPARK_USER") = sparkUser

    // We need to register "HeartbeatReceiver" before "createTaskScheduler" because Executor will
    // retrieve "HeartbeatReceiver" in the constructor. (SPARK-6640)
    // 心跳部分设置
    _heartbeatReceiver = env.rpcEnv.setupEndpoint(
      HeartbeatReceiver.ENDPOINT_NAME, new HeartbeatReceiver(this))

    // Create and start the scheduler
    val (sched, ts) = SparkContext.createTaskScheduler(this, master)
    _schedulerBackend = sched
    _taskScheduler = ts
    _dagScheduler = new DAGScheduler(this)
    _heartbeatReceiver.ask[Boolean](TaskSchedulerIsSet)

    // start TaskScheduler after taskScheduler sets DAGScheduler reference in DAGScheduler's
    // constructor
    _taskScheduler.start()

    //
    _applicationId = _taskScheduler.applicationId()
    _applicationAttemptId = taskScheduler.applicationAttemptId()
    _conf.set("spark.app.id", _applicationId)
    _ui.foreach(_.setAppId(_applicationId))
    _env.blockManager.initialize(_applicationId)

    // The metrics system for Driver need to be set spark.app.id to app ID.
    // So it should start after we get app ID from the task scheduler and set spark.app.id.
    metricsSystem.start()
    // Attach the driver metrics servlet handler to the web ui after the metrics system is started.
    metricsSystem.getServletHandlers.foreach(handler => ui.foreach(_.attachHandler(handler)))

    _eventLogger =
      if (isEventLogEnabled) {
        val logger =
          new EventLoggingListener(_applicationId, _applicationAttemptId, _eventLogDir.get,
            _conf, _hadoopConfiguration)
        logger.start()
        listenerBus.addListener(logger)
        Some(logger)
      } else {
        None
      }

    // Optionally scale number of executors dynamically based on workload. Exposed for testing.
    val dynamicAllocationEnabled = Utils.isDynamicAllocationEnabled(_conf)
    if (!dynamicAllocationEnabled && _conf.getBoolean("spark.dynamicAllocation.enabled", false)) {
      logWarning("Dynamic Allocation and num executors both set, thus dynamic allocation disabled.")
    }


    _executorAllocationManager =
      if (dynamicAllocationEnabled) {
        Some(new ExecutorAllocationManager(this, listenerBus, _conf))
      } else {
        None
      }
    _executorAllocationManager.foreach(_.start())

    _cleaner =
      if (_conf.getBoolean("spark.cleaner.referenceTracking", true)) {
        Some(new ContextCleaner(this))
      } else {
        None
      }
    _cleaner.foreach(_.start())

    setupAndStartListenerBus()
    postEnvironmentUpdate()
    postApplicationStart()

    // Post init
    _taskScheduler.postStartHook()
    _env.metricsSystem.registerSource(_dagScheduler.metricsSource)
    _env.metricsSystem.registerSource(new BlockManagerSource(_env.blockManager))
    _executorAllocationManager.foreach { e =>
      _env.metricsSystem.registerSource(e.executorAllocationManagerSource)
    }

    // Make sure the context is stopped if the user forgets about it. This avoids leaving
    // unfinished event logs around after the JVM exits cleanly. It doesn't help if the JVM
    // is killed, though.
    _shutdownHookRef = ShutdownHookManager.addShutdownHook(
      ShutdownHookManager.SPARK_CONTEXT_SHUTDOWN_PRIORITY) { () =>
      logInfo("Invoking stop() from shutdown hook")
      stop()
    }
  } catch {
    case NonFatal(e) =>
      logError("Error initializing SparkContext.", e)
      try {
        stop()
      } catch {
        case NonFatal(inner) =>
          logError("Error stopping SparkContext after init error.", inner)
      } finally {
        throw e
      }
  }
```

## REF

* [Spark源码学习（2）——Spark Submit](https://blog.csdn.net/sbq63683210/article/details/51638199)
* [Spark源码系列（一）spark-submit提交作业过程](http://www.cnblogs.com/cenyuhai/p/3775687.html)
* [源码-Spark on Yarn](https://blog.csdn.net/u011007180/article/details/52423483)
* [SparkSubmit启动应用程序主类过程](https://blog.csdn.net/ktlinker1119/article/details/45192575)
* [Spark源码解读之SparkContext初始化](https://blog.csdn.net/xw\_classmate/article/details/53408245)
* [Spark Storage之ExternalBlockStore](https://blog.csdn.net/u011564172/article/details/77947134)
