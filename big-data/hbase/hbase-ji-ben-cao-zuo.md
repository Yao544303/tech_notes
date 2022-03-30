# HBase基本操作

## HBase shell

### Hbase 创建表

```
create 'ych_test','cf'
```

### Hbase put

```
put 'ych_test','list1','cf:key','apple'
put 'ych_test','list1','cf:value','1'
put 'ych_test','list2','cf:key','apple'
put 'ych_test','list2','cf:value','2'
put 'ych_test','list3','cf:key','bad'
put 'ych_test','list3','cf:value','1'
put 'ych_test','list4','cf:key','bad'
put 'ych_test','list4','cf:value','2'
```

## Hbase 协处理器 编写记录

### 首先生成proto文件

protobuf 相关内容 自行google

```
option java_package = "com.zte.test.hbase";  
option java_outer_classname = "IPeopleScanner";  
option java_generic_services = true;  
option java_generate_equals_and_hash = true;  
option optimize_for = CODE_SIZE;  
  
message Condition {  
	required string jgsjStart =1;
	required string jgsjEnd = 2;
	required string pageSize = 3;
	required string pageNo = 4;
	optional string hphm = 5;
	optional string hpys = 6;
	repeated string dwbh = 7;
	repeated string qyfw = 8;
	optional string minClsd = 9;
	optional string maxClsd = 10;
	optianal string cdbh = 11;
}  

message ResultList { 
	required string total =1 
    repeated group Result = 2 {
		optional string rowKey = 1;
		optional string clxxbh = 2;
		optional string hphm = 3;
		optional string hpys = 4;
		optional string dwbh = 5;
		optional string jgsj = 6;
		optional string cdbh = 7;
		optional string clsd = 8;
		optional string xsfx = 9;
	}
}  
  
service Scan {  
    rpc getScanner(Condition)  
        returns (ResultList);  
}
```

其中

* required 是必要的，不可为空
* optional 是可选的
* repeat是可重复的

在协处理器中，要求每个result 都必须赋值，否则报错,每个可能用到的condition 都必须提前设置

```
protoc -I=$SRC_DIR --java_out=$DST_DIR $SRC_DIR/addressbook.proto
$SRC_DIR ：表示.proto文件所在目录；$DST_DIR：生成的java代码的文件夹。

例如
protoc.exe --java_out=./ CLGJXX.proto
```

### 协处理器 endpoint 的编写

继承四个方法

* start：用户获取当前env
* stop:
* getService： return this 即可
* getScan： 主要功能实现

### Hbase 协处理器启用

#### disable指定表。hbase> disable 'mytable'

#### 添加aggregation hbase>

```
alter 'vedio2human', METHOD => 'table_att','coprocessor'=>'|org.apache.hadoop.hbase.coprocessor.AggregateImplementation||'
alter 'video2car', METHOD => 'table_att','coprocessor' => 'file:///home/mr/hbase/lib/queryService-1.0.jar|com.zte.zxcloud.golddata.vap.query.vehicle.VehicleScanner|'

alter 'video2person', METHOD => 'table_att','coprocessor' => 'file:///home/mr/hbase/lib/queryService-1.0.jar|com.zte.zxcloud.golddata.vap.query.people.PeopleScanner|'
 
 alter 'video2car',METHOD=>'table_att_unset','coprocessor' => 'file:///home/mr/hbase/lib/vap-query.jar|com.zte.zxcloud.golddata.vap.query.vehicle.VehicleScanner|'
 
 alter 'video2car',METHOD => 'table_att_unset',NAME =>'coprocessor$1'
```

#### 重启指定表 hbase> enable 'mytable'

```
alter 'vedio2human', METHOD => 'table_att_unset', NAME => 'coprocessor$1'
enable 'tableName'

get 'vedio2human','starttime'
scan 'vedio2human',{COLUMNS =>'cf:sex',LIMIT =>10}
get 'vedio2human','612016-06-29 12:00:002a0cdec2-2dfe-4650-a298-99f80e0440b0'
```

## REF

* [apache hbase](https://hbase.apache.org)
* [HBase Scan 用法](http://www.cnblogs.com/cenyuhai/p/3231369.html)
* [HBase Scan 用法](http://www.cnblogs.com/linjiqin/archive/2013/06/05/3118921.html)
* [HBase shell](http://www.cnblogs.com/nexiyi/p/hbase\_shell.html)
