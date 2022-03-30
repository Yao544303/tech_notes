# HBase优化

## HBase 优化，columnFamily和qualifierColumn的设计原则

### 把一个传统的关系型数据库中的数据映射到hbase，从性能的角度如何优化ColumnFamily和qualifierColumn.

### 两个比较极端的情况

* 关系型数据库中的每一列对应一个columnFamily
* 关系型数据库中一张表对应一个columnFamily。

### 从读的角度分析性能

* 如果columnFamily越多，读取一个cell的速度优势是比较明显的，因为找到这个columnfamily，就等于找了column及其对应的值；
* 如果一张表对应一个columnfamily，找到对应的rowkey后，要把columnfamily对应的多列值都读取到，这样磁盘io和网络消耗的都比较多，速度会慢些。
* 如果某些列是经常要一起读取的，把这些归到一个columnfamily后面，一次请求就可以获取这些列，比分多次请求获取效率要高。

### 从写的角度分析性能

* 从regionserver内存消耗角度，根据hbase特点，一个columnfamily对应一个HStore,而每一个Hstore都有一个自己的memstore，如果columnfamily太多，对regionserver的内存消耗就很大。
* 从flush和compaction角度，目前hbase的flush和compaction都是以region为单位（虽然触发这个动作的条件有多个），如果columnfamily太多，很容易触发flush操作，对于很多memstore中的数据量可能还很少，这样flush就会产生大量小文件，而大量的小文件（即storefile）就会触发compaction操作，频繁的这样操作，会降低集群的性能。
* 从split角度分析，storefile是以columnfamily为单位的，大量的columnfamily可以减少split的发生，但这是一把双刃剑；因为的更少的split会导致部分region过于偏大，而regionserver之间进行balance时按region的数量进行负载均衡而不是按region的大小，这样可能就会导致balance失效。从好的一方面来看，更少的split会让集群运行的更稳定，然后选择在集群空闲或压力小的时候手动执行split和balance。
* 因此对于写部分，一般离线集群，一张表使用一个columnfamily即可，对于在线集群，可以根据情况合理分配columnfamily个数。

```
补充：目前我们的集群是在线集群，我们有一张在线使用表存储了很多数据，经过综合考虑只设计了一个columnfamily，主要考虑到，对于这个表中数据，每天查询量可以打三千万左右次，而表中每天新增数据只有几十G，这样设计可以减少split和flush的操作，让集群更多的时间处在稳定运行状态，这样有利于查询。
```

## REF

* [HBase 优化，columnFamily和qualifierColumn的设计原则](https://my.oschina.net/u/3197158/blog/994898/)
* [hbase建表create高级属性 //hbase 表预分区也就是手动分区 这个很重要](https://blog.51cto.com/12445535/2351994)
