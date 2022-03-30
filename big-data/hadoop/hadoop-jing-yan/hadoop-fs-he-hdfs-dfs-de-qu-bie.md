# hadoop fs和hdfs dfs的区别

## FS Shell

调用文件系统(FS)Shell命令应使用 bin/hadoop fs args 的形式。 所有的的FS shell命令使用URI路径作为参数。URI格式是scheme://authority/path。 对HDFS文件系统，scheme是hdfs，对本地文件系统，scheme是file。其中scheme和authority参数都是可选的，如果未加指定，就会使用配置中指定的默认scheme。一个HDFS文件或目录比如/parent/child可以表示成hdfs://namenode:namenodeport/parent/child，或者更简单的/parent/child（假设你配置文件中的默认值是namenode:namenodeport）。大多数FS Shell命令的行为和对应的Unix Shell命令类似，不同之处会在下面介绍各命令使用详情时指出。出错信息会输出到stderr，其他信息输出到stdout。 对于配置好的情况，比如我的用户为ych，对应hdfs 上的用户目录为 /user/ych/ 以下三个路径等价

* hdfs：//namenode:namenodeport/user/ych/a
* /user/ych/a
* a

## hadoop fs 和 hdfs dfs 的区别

个人因为接触hdfs 这个比较早，所以使用的都是hdfs dfs，直到有一天，看到霞姐敲出了hadoop fs 才打开了新世界的大门。那么这两者有什么区别呢？ 查看下 $HADOOP\_HOME/bin/hadoop

```
elif [ "$COMMAND" = "fs" ] ; then
  CLASS=org.apache.hadoop.fs.FsShell
  HADOOP_OPTS="$HADOOP_OPTS $HADOOP_CLIENT_OPTS"
elif [ "$COMMAND" = "dfs" ] ; then
  CLASS=org.apache.hadoop.fs.FsShell
  HADOOP_OPTS="$HADOOP_OPTS $HADOOP_CLIENT_OPTS"
```

恩，其实没区别，连文档中的[说明](https://dzone.com/articles/difference-between-hadoop-dfs)都类似 细致一点就是，使用功能的范围不同

### hadoop fs

使用面最广，可以操作任何文件系统 比如，Local FS, HFTP FS，S3 FS

### hdfs dfs

使用面比较小，只能操作hdfs 以及 hdfs 和 Local FS 交互 tips： 你可以试下 hdfs dfs -ls file:///

## ref

* [Hadoop Shell命令](http://hadoop.apache.org/docs/r1.0.4/cn/hdfs\_shell.html)
* ['hadoop fs'和'hdfs dfs'的区别](http://blog.csdn.net/pipisorry/article/details/51340838)
