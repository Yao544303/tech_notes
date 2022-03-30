# Hadoop 输入路径过滤

## 丢失文件过滤

我们想查询一个月以来度假的订单数据，但是HDFS中可能因为业务故障，导致某一天的订单数据不存在：

所以在设置路径之前需要进行一次判断，判断这个路径在HDFS上是否存在，如果存在，使用addInputPath方法添加：

```
FileSystem fileSystem = FileSystem.get(conf);
String[] params = inputPath.split(",");
for(String path : params){
   boolean isExists = fileSystem.exists(new Path(path));
   if(isExists){
      FileInputFormat.addInputPath(job, new Path(path));
   }
}
```

## 文件名过滤

在一步操作中处理批量文件，这个要求很常见。举例来说，处理日志的MapReduce作业可能会分析一个月的文件，这些文件被包含在大量目录中。Hadoop有一个通配的操作，可以方便地使用通配符在一个表达式中核对多个文件，不需要列举每个文件和目录来指定输入。Hadoop为执行通配提供了两个FileSystem方法：

```
public FileStatus[] globStatus(Path pathPattern) throws IOException
public FileStatus[] globStatus(Path pathPattern, PathFilter filter) throws IOException
```

globStatus()返回了其路径匹配于所供格式的FileStatus对象数组，按路径排序。可选的PathFilter命令可以进一步指定限制匹配。

### Hadoop支持的一系列通配符与Unix bash相同

```
通配符	名称	匹配
*	星号	匹配0或多个字符
?	问号	匹配单一字符
[ab]	字符类别	匹配{a,b}中的一个字符
[^ab]	非字符类别	匹配不是{a,b}中的一个字符
[a-b]	字符范围	匹配一个在{a,b}范围内的 字符(包括ab)，a在字典 顺序上要小于或等于b
[^a-b]	非字符范围	匹配一个不在{a,b}范围内 的字符(包括ab)，a在字 典顺序上要小于或等于b
{a,b}	或选择	匹配包含a或b中的一个的语句
\c	转义字符	匹配元字符c
```

### PathFilter过滤

通配格式不是总能够精确地描述我们想要访问的文件集合。比如，使用通配格式排除一个特定的文件就不太可能。FileSystem中的listStatus()和globStatus()方法提供了可选的PathFilter对象，使我们能够通过编程方式控制匹配：

## REF

[Ref](https://blog.csdn.net/SunnyYoona/article/details/53786397)
