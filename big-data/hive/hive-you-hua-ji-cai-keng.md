# Hive 优化及踩坑



## 子查询要加别名，否则会报错

## 跨库查询需要加别名

## hvie常用函数

[hive常用函数](http://blackproof.iteye.com/blog/2108353)

## hive不支持 视图的union 要小心

## notin

写了好几个页面，速度都上不去，瓶颈在于SQL查询。太多的表，太多的not in，总是从一大推表和数据中筛选出一点数据。看了很多关于SQL优化的文章，都强烈要求不要太多使用not in查询，最好用表连接来取代它。如：

```
select ID,name from Table_A where ID not in (select ID from Table_B)
```

这句是最经典的not in查询了。改为表连接代码如下：

```
select Table_A.ID,Table_A.name from Table_A left join Table_B on Table_A.ID=Table_B.ID and Table_B.ID is null
或者：
select Table_A.ID,Table_A.name from Table_A left join Table_B on Table_A.ID=Table_B.ID where Table_B.ID is null
```
