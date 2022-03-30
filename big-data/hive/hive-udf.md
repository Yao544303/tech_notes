# Hive UDF

### 过程

第一步：继承UDF，实现evaluate()方法，方法的参数和返回类型自己定义。 第二步：将写好的类打包成jar文件，如myhive.jar。 第三步：进入到Hive终端，利用add jar path/myhive.jar将自定义的jar加入到Hive环境中。 第四步：为该类起一个别名，create temporary function toUpper as 'com.lixue.udf.UDFToUpper'。 第五步：在select语句中使用toUpper()方法。

### 添加到hive 环境

```
add jar /data/users/ych/bigdata-encryption-0.0.1-SNAPSHOT.jar;
```

### 创建临时函数

```
create temporary function encoding as 'com.xxx.encryption.EncryptionUDF';  
```

### 使用

```
select encoding('xxxxxxxxx');
```

### 删除临时函数

```
drop temporary function encoding;
```

### 注：

UDF只能实现一进一出的操作，如果需要实现多进一出，则需要通过UDAF来实现。

### REF

* [Hive自定义UDF](http://blog.csdn.net/lzm1340458776/article/details/43311797)
