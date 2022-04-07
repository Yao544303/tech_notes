# Shell 中for 循环的几种写法

# 第一类 数字型循环
注意，表达式expr 中，运算符和参数之间必须空格，否则报错，比如这种  
```
expr $i \* 3 +1
```

##  for1-1.sh
```
#!/bin/bash
for ((i=1;i<10;i++))
do
    echo $(expr $i \* 3 + 1);
done
```

## for1-2.sh
```
#!/bin/bash
for i in$(seq 1 10)
do 
    echo $(expr $i \* 3 + 1);
done
```

## for1-3.sh
```
#!/bin/bash
for i in {1..10}
do
    echo $(expr $i \* 3 + 1);
done
```


## for1-4.sh
```
awk 'BEGIN{for(i=1;i<=10;i++) print i}'
```

# 第二类 字符型循环
## for2-1.sh
```
#!/bin/bash  
for i in `ls`;  
do   
    echo $i is file name\! ;  
done
```

## for2-2.sh
```
#!/bin/bash  
for i in $* ;  
do  
    echo $i is input chart\! ;  
done
```

## for2-3.sh
```
#!/bin/bash
for i in f1 f2 f3;
do 
    echo $i is appoint;
done
```

## for2-4.sh
```
#!/bin/bash
list="rootfs usr data data2"
for i in $list;
do
    echo $i is appoint;
done
```

## 第三类 路径查找
```
#!/bin/bash
for file in /proc/*
do
    echo $file is file path \!;
done
```

```
#!/bin/bash
for file in $(ls *.sh)
do
    echo $file is file path \!;
done
```