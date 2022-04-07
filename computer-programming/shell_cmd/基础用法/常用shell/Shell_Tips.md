# 目录部分
## 判断目录是否存在，不存在则新建一个目录
```
DIR="testDIR"
if [ !-d "$DIR"]
then mkdir "$DIR"
fi
```
## mkdir & cd
```
mkdir "my directory" && cd $_
```

## 获取当前目录1 
脚本用于获取脚本所在目录的路径可以解决大多数问题，缺陷是对于软链接显示的是软链接所在的目录
```
DIR="$( cd "$( dirname "$0"  )" && pwd  )"
echo ${DIR}
```


## 获取当前目录2
 这个版本解决了使用ln -s target linkName创造软链接无法正确取到真实脚本的问题。
```
SOURCE="$0"
while [ -h "$SOURCE"  ]; do # resolve $SOURCE until the file is no longer a symlink
    DIR="$( cd -P "$( dirname "$SOURCE"  )" && pwd  )"
    SOURCE="$(readlink "$SOURCE")"
    [[ $SOURCE != /*  ]] && SOURCE="$DIR/$SOURCE" # if $SOURCE was a relative symlink, we need to resolve it relative to the path where the symlink file was located
done
DIR="$( cd -P "$( dirname "$SOURCE"  )" && pwd  )"
echo $DIR
```


# 日期
## 获取今天的时间
```
DATE=`date +%Y%m%d`
echo ${DATE}

DATE=`date +%F`
echo ${DATE}

DATE=`date +%y%m%d`
echo ${DATE}
```

## 获取昨天的日期
```
YESTERDAY=`date -d yesterday +%Y%m%d`
```

## 前天
```
date -d "2 days ago" +%Y%m%d  
```

## n天前
```
date -d "n days ago" +%Y%m%d
```

## 日期格式
```
%Y%m%d
%Y-%m-%d
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

# 文件操作
## 删除文件中含特定字符串的行[bash]:
```
sed -e '/abc/d'  a.txt   // 删除a.txt中含"abc"的行，但不改变a.txt文件本身，操作之后的结果在终端显示

sed -e '/abc/d'  a.txt  > a.log   // 删除a.txt中含"abc"的行，将操作之后的结果保存到a.log

sed '/abc/d;/efg/d' a.txt > a.log    // 删除含字符串"abc"或“efg"的行，将结果保存到a.log
```
其中，"abc"也可以用正则表达式来代替。

## 列数统计
cat 初筛规则.txt|awk -F'~' '{print NF}'|sort|uniq -c

## 去除第二列 加指定编码
```
for i in {01..08}
do
awk -F'|' '{print $1}' data${i} |  sed 's/$/|'$i'/' > data.group${i}
#cat data.g${i} | sed 's/$/|'$i'/' > data.group${i}
done
```

## 第一列求和
```
cat file | awk '{print $1}' | awk '{sum+=$1}END{print sum}'
```

## 在文件开始或者结尾插入一列字符串，采用sed更容易实现：
```
sed  's/^/hello world &/g' data
sed 's/$/& hello world/g' data
```
## awk根据|分割输出第一列
```
awk -F'|' '{print $1"|"$6}' 
```


## 比较两个文件的差异（DOS）
```
FC file1 file2	 
```

# 其他小技巧
## 在命令前，加个时间戳，便于辨认
```
echo `date "+%Y-%m-%d %H:%M:%S"` ${CMD} >>log 
${CMD}  >>log  2>&1

echo `date "+%Y-%m-%d %H:%M:%S"` ${CMD} |tee -a log 
${CMD} 2>&1 |tee -a log 
```

## 显示进程的启动位置  
```
pwdx
```

## 输出环境变量（DOS）
```
echo %path%
```

## 查看jar包内容
```
jar vtf  fileName.jar
```

