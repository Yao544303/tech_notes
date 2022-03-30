# SparkSubmit源码走读3\_SparkEnv

## 创建SparkEnv

```
private[spark] def createSparkEnv(
    conf: SparkConf,
    isLocal: Boolean,
    listenerBus: LiveListenerBus): SparkEnv = {
  SparkEnv.createDriverEnv(conf, isLocal, listenerBus, SparkContext.numDriverCores(master))
}
```

调用了org.apache.spark.SparkEnv.createDriverEnv 校验是否设置了spark.driver.host 和 spark.driver.port(在sparkContext初始化时，为空会置为local:0),并获取这两个值

```
-----> create 
        创建securityManager 校验用户信息，change view 和 mode
        创建 actorSystemName 变量，根据当前节点是否是driver 设置其值
        创建rpcEnv 原型 create(actorSystemName: String, hostname: String, port: Int, conf: SparkConf, securityManager: SecurityManager,clientMode: Boolean)
            -> 根据入参，创建RpcEnvConfig,调用getRpcEnvFactory 在 netty 和 akka中二选一,调用Utils.classForName(这一部分需要进一步理解) 创建RpcEnvFactory
                -> 调用RpcEnvFactory 的create方法返回RpcEnv
        根据rpcEnv 创建 ActorSystem  -> 根据条件，先在 AkkaRpcEnv 中调用AkkaUtils 再直接调用 AkkaUtils  (这一部分理解有问题，需要更近一步)
            -> createActorSystem
        设置spark.driver.port spark.executor.port 为 akka端口      
        指定序列化类 val serializer = instantiateClassFromConf[Serializer]，在debug日志等级下，这里也会打出日志
        注册mapOutputTracker:主要就是跟踪map任务把数据结果写到哪里去了，reduce也可以去取数据map，reduce，shuffle都有自己所对应的ID，着重介绍一下    MapOutputTrackerMaxter，它内部使用mapStatuses来维护跟中map任务的输出状态，这个数据结构是一个map，其中key对应shufleID，value存储各个map任务对应的状态信息mapStatus。由于mapStatus是由blockmanagerid(key)和bytesSize(value)组成，key是表示这些计算的中间结果存放在那个blockManager，value表示不同的reduceID要读取的数据大小这时reduce就知道从哪里fetch数据，并且判断数据的大小(和0比较来确保确实获得了数据)。driver和executor处理mapOutputtTrackermMaster的方式不同: Driver:创建mapOutputtTrackermMaster,然后创建mapOutputtTrackermMasterActor,并且注册到ActorSystem.Executor:创建mapOutputtTrackerm，并从ActorSystem中找到mapOutputtTrackermMasterActor。
        注册BlockManagerMaster 它是负责block的管理和协调，具体的操作是依赖于BlockManagerMaster-Actor
        创建BlockManger 他主要运行在worker节点上。虽然这里创建了，但是只有在他的init初始化函数之后才是有效的
        创建broadcastManager 主要是负责把序列化时候的RDD，job以及shuffleDependence等，以及配置信息存储在本地，有时候还会存储到其他的节点以保持可靠性。
        创建cacheManager 主要用于缓存RDD某个分区计算的中间结果，缓存计算结果在迭代计算的时候发生。他很有用，可以减少磁盘IO，加快执行速度。
        创建metricsSystem spark的测量系统
        创建sparkFileDir
        创建outputCommitCoordinator
        创建outputCommitCoordinatorRef
        （至此结束，返回SparkEnv）
```
