# Hive 常用操作

## 使用beeline连接

!connect jdbc:hive2://servernode1:10000/default

## 添加分区

```
alter table t3_sl_rules add partition (day='20180628',product='360',province_code='33',rule_id='S0002');
```

## 修复分区

```
MSCK REPAIR TABLE table_name;
```

## insert overwrite

```
insert overwrite directory 'hdfs://localhost:9000/app/test/' select concat(userid,"\t",mdt,"\t",case when loginfo['id'] is null then '' else loginfo['qid'] end,"\t" ,trim(urlpath),"\t",fr,"\t",loginfo['pro_errno'],"\t",province,"\t",city) from testapi where dt='20151025'

insert overwrite local directory '/home/test/data/result'
  row format delimited
  fields terminated by '\t'
  select * from test;
```

## 创建外表并跳过头尾

```
CREATE EXTERNAL TABLE db.table_name (
	d_brand		STRING,
	domain		STRING,
	user_id		STRING,
	hour		INT,
	content_len INT,
	outout_octets INT,
	d_model 	STRING,
	os_version 	STRING,
	content_type STRING,
	visit_port 	INT,
	device		STRING,
	input_octets INT,
	os			STRING,
	d_type		STRING,
	b_version	STRING,
	browser		STRING,
	user_type	INT,
	ds			STRING
)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\,'
LOCATION
  'hdfs://nameservice/user/ych.db/table1'
 TBLPROPERTIES (
 "skip.header.line.count"="1",
 "skip.footer.line.count"="2");
```

## 使用avro文件创建表

```
CREATE EXTERNAL TABLE "db.t1_tt"
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.avro.AvroSerDe'
STORED AS
INPUTFORMAT 'org.apache.hadoop.hive.ql.io.avro.AvroContainerInputFormat'
OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.avro.AvroContainerOutputFormat'
LOCATION 'hdfs://nameservice/group/bigdata_dev_dashuju/xxx'
TBLPROPERTIES (
'avro.schema.literal'='{"namespace": "idp.model",
 "type": "record",
 "name": "DtDayInfo",
 "fields": [
     {"name": "user_id", "type": "string"},
     {"name": "day", "type": "string"},
     {"name": "domain",  "type": "string"},
     {"name": "site",  "type": "string"},
     {"name": "first_category", "type": "string"},
     {"name": "second_category", "type": "string"},
     {"name": "third_category", "type": "string"},
     {"name": "source", "type": "string"},
	 {"name": "percent", "type": "string"},
     {"name": "frequency", "type": "int"},
     {"name": "city_code", "type":["string", "null"], "default": "null"},
	 {"name" : "extract", "type" : [{
        "type" : "array",
        "items" : {
            "type" : "record",
            "name" : "ValuePair",
            "fields" : [
                {"name" : "rule_id","type":["string", "null"], "default": "null"},
                {"name" : "name", "type":["string", "null"], "default": "null"},
                {"name" : "value", "type":["string", "null"], "default": "null"}
            ]}
        },
        "null"
    ],"default": "null"}
 ]
}'
);
```

## 使用parquet 文件建表

注意复杂结构 array。多维array暂不支持

```
CREATE EXTERNAL TABLE `db.e_talkingdata3`(
	appId bigint,
	activeChangeRate DOUBLE,
	appIconUrl STRING,
	coverageChangeRate bigint,
	newApp boolean,
	rankingChange bigint,
	appName STRING,
	appNameEn STRING,
	company STRING,
	companyEn STRING,
	id bigint,
	platform bigint,
	types Array<struct<typeId:bigint,typeName:STRING>>,
	active array<bigint>,
	activePerCoverage array<DOUBLE>,
	activeRate array<DOUBLE>,
	activeRateBenchmarkH array<DOUBLE>,
	activeRateBenchmarkL array<DOUBLE>,
	activeRateChange array<STRING>,
	aspu array<bigint>,
	avgDau array<bigint>,
	avgDauRate array<bigint>,
	coverage array<bigint>,
	coverageRate array<DOUBLE>,
	coverageRateBenchmarkH array<DOUBLE>,
	coverageRateBenchmarkL array<DOUBLE>,
	coverageRateChange array<STRING>,
	date array<STRING>,
	ranking array<bigint>,
	typeRanking array<bigint>,
	mau array<struct<code:STRING,name:STRING,share:STRING>>,
	gender array<struct<code:STRING,name:STRING,share:STRING>>,
	age array<struct<code:STRING,name:STRING,share:STRING>>,
	province  array<struct<code:STRING,name:STRING,share:STRING>>,
	city  array<struct<code:STRING,name:STRING,share:STRING>>,
	consumption  array<struct<code:STRING,name:STRING,share:STRING>>,
	preference  array<struct<code:STRING,name:STRING,share:STRING>>
)
ROW FORMAT SERDE 'parquet.hive.serde.ParquetHiveSerDe'
 STORED AS
 INPUTFORMAT 'parquet.hive.DeprecatedParquetInputFormat'
 OUTPUTFORMAT 'parquet.hive.DeprecatedParquetOutputFormat';
LOCATION
  'hdfs://nameservice/group/db/files/gom_bak/talkingdata'
```

## 查询结果插入

### 一般方式：

```
insert overwrite directory '/hive/test_data' 
select * from test; 
```

### 自定义输出样式方式：

```
insert overwrite directory '/hive/test_data' row format delimited fields terminated by ',' 
select * from test; 
```

## 查询结果导入到本地文件

### 一般方式：

```
insert overwrite local directory '/hive/test_data' 
select * from test;
```

### 自定义输出样式方式：

```
insert overwrite local directory '/hive/test_data' row format delimited fields terminated by ',' 
select * from test;
```

## 分区表插入

```
insert overwrite table bigdata_fibodt.e_gender_unknown PARTITION (p_province) select v0To2(mobile) as uid, gender,probabilty,p_province from bigdata_fibodt.e_gender_unknown join dw_resources.mapping_md5_property on uid = md5md5 where p_operate='0'
```

## hive表导入CSV数据

```
 create table IP(ip varchar(30), country varchar(30), province varchar(30), city varchar(30), district varchar(30), linetype varchar(30))
 row format delimited fields terminated by ',' ;
```

然后再输入导入的语句：

```
load data local inpath '/usr/testFile/result.csv' overwrite into table biao;
（load data local inpath '文件路径' overwrite into table 表名;）
```

然后最后查询

```
show tables;

select * from ip limit 100;
```
