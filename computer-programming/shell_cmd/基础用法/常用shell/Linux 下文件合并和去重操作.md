Linux 下文件合并和去重操作

# 注意
1. 进行操作之后，无法直接重定向到原文件

# 涉及命令
```
cat

uniq

sort

paste
```

# 实例应用
## 两个文件求交、并集（前提条件，每个文件中不得有重复）
1. 去除两个文件的并集  
```
$ cat file1 file2 | sort | uniq > file3

2. 取出两个文件的交集  
$ cat file1 file2 | sort | uniq -d > file3

3. 删除交集，留下其他的行  
$ cat file1 file2 | sort | uniq -u > file3

4. 两个文件去重
sort a.data b.data b.data. | uniq -u > a.result     这是a-b
```
## 两个文件合并
```
1. 一个文件在上，一个文件在下  
$ cat file1 file2 > file3

2. 一个文件在左，一个文件在右  
$ paste file1 file2 > file3
```
## 一个文件去掉重复的行
```
1. 重复的多行记为一行  
$ sort file | uniq

2. 重复的行全部去掉  
$ sort file | uniq -u 

3. 使用uniq/sort删除重复行
shell> sort -k2n file | uniq > a.out

4. 使用用sort+awk命令
shell> sort -k2n file | awk '{if ($0!=line) print;line=$0}'

5. 用sort+sed命令，同样需要sort命令先排序。
shell> sort -k2n file | sed '$!N; /^\(.*\)\n\1$/!P; D'
```

## 随机取数
```
sort --random-sort dw_sh_pos_201801_678.txt | head -n 10000 > dw_sh_pos_201801_678s.txt
```

# REF
* [shell中删除文件中重复行的方法](http://www.jb51.net/article/48077.htm)