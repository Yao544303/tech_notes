# Spark配置（conf)

## Spark 配置

Spark 配置控制着大多数应用的设置，并且可根据每个应用单独设置。这些配置项可以通过SparkConf，按照键-值对的方式直接设置。 配置项设置持续时间时可以使用一些时间单位，如下这些都是可以被识别的:

```
25ms (milliseconds)
5s (seconds)
10m or 10min (minutes)
3h (hours)
5d (days)
1y (years)
```

如下这些空间大小的计量单位也是可以被识别的。

```
1b (bytes)
1k or 1kb (kibibytes = 1024 bytes)
1m or 1mb (mebibytes = 1024 kibibytes)
1g or 1gb (gibibytes = 1024 mebibytes)
1t or 1tb (tebibytes = 1024 gibibytes)
1p or 1pb (pebibytes = 1024 tebibytes)
```

## 动态加载Spark 配置

在一些案例中，你可能想避免hard-coding这些参数，比如你想在不同的master或者内存上上运行同一段spark程序。Spark允许你创建一个空的conf，在submit的时候再设置参数。

## 查看Spark 配置

在应用的web UI @http://:4040 列出了Spark的参数。这是一个非常实用的地方来保证你的参数配置正确。注意，只有通过spark-defaults.conf,SparkConf或者命令行设定的参数才能在这里显示。对于其他没有显示的参数，你可以认为使用的是默认值。

## 可用的配置

大多数控制内部设定的配置都有很合理的默认值。一些常用的选项设置如下：

###
