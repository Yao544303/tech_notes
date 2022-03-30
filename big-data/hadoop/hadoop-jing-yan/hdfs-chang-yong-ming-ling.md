# hdfs常用命令

## HDFS 常用命令

### 文件操作

以下操作都在目录/usr/local/hadoop 下 列出HDFS下的文件

```
hadoop dfs -ls
```

列出HDFS文件下名为in的文档中的文件

```
hadoop dfs -ls in
```

上传文件 将hadoop目录下test1 文件上传到HDFS上并重命名为test

```
hadoop dfs -put test1 test
```

文件被复制到本地系统中，将HDFS中的in文件复制到本地系统并命名为getin：

```
hadoop dfs -get in getin
```

删除文档 删除HDFS下名为out的文档

```
hadoop dfs -rmr out
```

查看文件 查看HDFS下in文件中的内容

```
hadoop dfs -cat in/*
```

建立目录

```
hadoop dfs -mkdir /usr/hadoop
```

复制文件

```
hadoop dfs -copyFromlocal 源路径 路径
```

通过hadoop命令把两个文件的内容合并起来 注：合并后的文件位于当前目录，不在hdfs中，是本地文件

```
hdsf dfs -getmerge hdfs://Master:9000/data/SogouResult.txt CombinedResult
```

查看hdfs目录下所有文件的大小

```
hdfs  dfs -du -s -h tmp/ych
```

### 管理与更新

执行基本信息 查看HDFS的基本统计信息

```
hadoop dfsadmin -report
```

退出安全模式 NameNode在启动时会自动进入安全模式。安全模式是NameNode的一种状态，在这个阶段，文件系统不允许有任何修改。 系统显示Name node in safe mode，说明系统正处于安全模式，这时只需要等待十几秒即可，也可使用上述命令退出安全模式。

```
hadoop dfsadmin -safemode leave
```

进入安全模式

```
hadoop dfsadmin -safemode enter
```

负载均衡：HDFS的数据在各个DataNode 中的分布可能不均匀，尤其在DataNode节点出现故障或新增DataNode节点时。

```
start-balancer.sh
```

### 统计

统计文件大小

```
hadoop fs -du -h 
```

统计文件数量，返回的数据是目录个数，文件个数，文件总计大小，输入路径

```
hadoop fs -count 
```

### 其他

chgrp 改变文件所属的用户组

```
hadoop dfs -chgrp [-R] GROUP URI [URI ...]
```

## REF

* [Hadoop Shell命令](http://hadoop.apache.org/docs/r1.0.4/cn/hdfs\_shell.html)
