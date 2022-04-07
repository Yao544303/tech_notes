# 清空文件内容

### 1.通过 shell 重定向 null （不存在的事物）到该文件：

```
  > file
```

### 2.使用 ‘true' 命令重定向来清空文件

```
: > access.log
true > access.log
```

### 3.使用 cat/cp/dd 实用工具及 /dev/null 设备来清空文件

```
cat /dev/null > access.log
cp /dev/null access.log
dd if=/dev/null of=access.log
```

### 4.使用 echo 命令清空文件

```
echo "" > access.log
echo > access.log
```

### 5.使用 truncate 命令来清空文件内容

```
truncate -s 0 access.log
```

## REF

* [ref](http://www.jb51.net/article/100462.htm)
