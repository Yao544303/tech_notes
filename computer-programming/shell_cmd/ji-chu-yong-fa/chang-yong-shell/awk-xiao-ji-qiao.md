# awk 小技巧

## 统计

有文件file.log内容如下：

```
http://www.sohu.com/aaa
http://www.sina.com/111
http://www.sohu.com/bbb
http://www.sina.com/222
http://www.sohu.com/ccc
http://www.163.com/zzz
http://www.sohu.com/ddd
```

要统每个域名出现次数：

```
http://www.sohu.com 4
http://www.sina.com 2
http://www.163.com 1
```

答案是：

```
awk -F / '{a[$3]++} END{for(i in a){print i,a[i] | "sort -r -k 2"}}' file.log;
```

解释一下，awk语法就不说了： -F参数是制定awk分隔符，这里制定的是 /,所以每行被分成4个部分。 sort 的-r是降序，-k是按照第几组字符排序，从1开始。 a可以理解成key-value形式的对象，域名做key 个数做value。 在end动作里完成对结果a的打印

## 数学计算+统计

```
cat gender_predict|awk -F'|' '{print int($2/0.05)}' | sort | uniq -c
```

## REF

* [REF](http://blog.sina.com.cn/s/blog\_520fb00d0100om3v.html)
