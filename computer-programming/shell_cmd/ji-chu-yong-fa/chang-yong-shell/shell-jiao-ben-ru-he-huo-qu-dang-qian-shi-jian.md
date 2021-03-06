# shell脚本如何获取当前时间

## 获取当天日期

```
[root@centi-C sh]# date +%Y%m%d 
20120727 
[root@centi-C sh]# date +%F 
2012-07-27 
[root@centi-C sh]# date +%y%m%d 
120727
```

## 获取昨天日期

```
[root@centi-C sh]# date -d yesterday +%Y%m%d 
20120726 
[root@centi-C sh]# date -d yesterday +%F 
2012-07-26 
[root@centi-C sh]# date -d -1day +%y%m%d 
120726 
[root@centi-C sh]# date -d -1day +%Y%m%d 
20120726 
```

## 获取前天日期

```
`date -d -2day +%Y%m%d` 
```

以此类推

## 格式

至于你需要什么样的日期时间格式，就需要应用相关的时间域参数来实现咯 相关时间域如下：

```
% H 小时（00..23） 
% I 小时（01..12） 
% k 小时（0..23） 
% l 小时（1..12） 
% M 分（00..59） 
% p 显示出AM或PM 
% r 时间（hh：mm：ss AM或PM），12小时 
% s 从1970年1月1日00：00：00到目前经历的秒数 
% S 秒（00..59） 
% T 时间（24小时制）（hh:mm:ss） 
% X 显示时间的格式（％H:％M:％S） 
% Z 时区 日期域 
% a 星期几的简称（ Sun..Sat） 
% A 星期几的全称（ Sunday..Saturday） 
% b 月的简称（Jan..Dec） 
% B 月的全称（January..December） 
% c 日期和时间（ Mon Nov 8 14：12：46 CST 1999） 
% d 一个月的第几天（01..31） 
% D 日期（mm／dd／yy） 
% h 和%b选项相同 
% j 一年的第几天（001..366） 
% m 月（01..12） 
% w 一个星期的第几天（0代表星期天） 
% W 一年的第几个星期（00..53，星期一为第一天） 
% x 显示日期的格式（mm/dd/yy） 
% y 年的最后两个数字（ 1999则是99） 
% Y 年（例如：1970，1996等） 
```

## 遍历输出两个日期范围内所有日期的方法

```
START=20190620
END=20190629
while(("${START}" <= "${END}"));do
CUR=$(date -d "${START} +1day" +%Y%m%d)
echo ${CUR}
START=${CUR}
done
```

## REF

* [shell脚本如何获取当前时间](https://blog.csdn.net/nowayings/article/details/48492497)
* [shell通过遍历输出两个日期范围内所有日期的方法](https://www.jb51.net/article/116538.htm)
