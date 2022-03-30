# Dataset API的优势

## Dataset API的优势

对于Spark开发者而言，你将从Spark 2.0的DataFrame和Dataset统一的API获得以下好处：

### 静态类型和运行时类型安全

考虑静态类型和运行时类型安全，SQL有很少的限制而Dataset限制很多。例如，Spark SQL查询语句，你直到运行时才能发下语法错误（syntax error），代价较大。然后DataFrame和Dataset在编译时就可捕捉到错误，节约开发时间和成本。 Dataset API都是lambda函数和JVM typed object，任何typed-parameters不匹配即会在编译阶段报错。因此使用Dataset节约开发时间。

### High-level抽象以及结构化和半结构化数据集的自定义视图

DataFrame是Datasets\[Row]的特例，把结构化数据集视图用于半结构化数据集。例如，有个海量IoT设备事件数据集，用JSON格式表示。JSON是一个半结构化数据格式，这里可以自定义一个Dataset：Dataset\[DeviceIoTData]。

```
{"device_id": 198164, "device_name": "sensor-pad-198164owomcJZ", "ip": "80.55.20.25", "cca2": "PL", "cca3": "POL", "cn": "Poland", "latitude": 53.080000, "longitude": 18.620000, "scale": "Celsius", "temp": 21, "humidity": 65, "battery_level": 8, "c02_level": 1408, "lcd": "red", "timestamp" :1458081226051}
```

用Scala为JSON数据DeviceIoTData定义case class。

```
case class DeviceIoTData (battery_level: Long, c02_level: Long, cca2: String, cca3: String, cn: String, device_id: Long, device_name: String, humidity: Long, ip: String, latitude: Double, lcd: String, longitude: Double, scale:String, temp: Long, timestamp: Long)
```

紧接着，从JSON文件读取数据：

```
// read the json file and create the dataset from the 
// case class DeviceIoTData
// ds is now a collection of JVM Scala objects DeviceIoTData

val ds = spark.read.json("/databricks-public-datasets/data/iot/iot_devices.json").as[DeviceIoTData]
```

这个时候有三个事情会发生：

* Spark读取JSON文件，推断出其schema，创建一个DataFrame；
* Spark把数据集转换DataFrame = Dataset\[Row]，泛型Row object，因为这时还不知道其确切类型；
* Spark进行转换：Dataset\[Row] -> Dataset\[DeviceIoTData]，DeviceIoTData类的Scala JVM object。

### 简单易用的API

虽然结构化数据会给Spark程序操作数据集带来挺多限制，但它却引进了丰富的语义和易用的特定领域语言。大部分计算可以被Dataset的high-level API所支持。例如，简单的操作 agg，select，sum，avg， map，filter 或者 groupBy 即可访问DeviceIoTData类型的Dataset。 使用特定领域语言API进行计算是非常简单。例如，使用\*filter()\*和 \*map()\*创建另外一个Dataset。

## 使用DataFrame和Dataset API获得空间效率和性能优化的两个原因：

* 首先，DataFrame和Dataset API是建立在Spark SQL引擎之上，它会使用Catalyst优化器来生成优化过的逻辑计划和物理查询计划。R， Java，Scala或者Python的DataFrame/Dataset API使得查询都进行相同的代码优化以及空间和速度的效率提升。
* 其次，Spark作为编译器可以理解Dataset类型的JVM object，它能映射特定类型的JVM object到Tungsten内存管理，使用Encoder。Tungsten的Encoder可以有效的序列化/反序列化JVM object，生成字节码来提高执行速度。

## 什么时候使用DataFrame或者Dataset？

* 你想使用丰富的语义，high-level抽象，和特定领域语言API，那你可DataFrame或者Dataset；
* 你处理的半结构化数据集需要high-level表达， filter，map，aggregation，average，sum ，SQL 查询，列式访问和使用lambda函数，那你可DataFrame或者Dataset；
* 你想利用编译时高度的type-safety，Catalyst优化和Tungsten的code生成，那你可DataFrame或者Dataset；
* 你想统一和简化API使用跨Spark的Library，那你可DataFrame或者Dataset；
* 如果你是一个R使用者，那你可DataFrame或者Dataset；
* 如果你是一个Python使用者，那你可DataFrame或者Dataset；
