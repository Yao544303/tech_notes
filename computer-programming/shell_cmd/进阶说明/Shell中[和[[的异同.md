1.     概念上来说

"[["，是关键字，许多shell(如ash bsh)并不支持这种方式。ksh, bash(据说从2.02起引入对[[的支持)等支持。
"["是一条命令， 与test等价，大多数shell都支持。在现代的大多数sh实现中，"["与"test"是内部(builtin)命令，换句话说执行"test"/"["时不会调

2.     相同：二者都支持算术比较和字符串比较表达式（具体使用可能有点不同）

（1）"-gt", "-lt"是算术比较操作符，用于比较整数的大小。
（2）">", "<"是字符串比较操作符，用于比较字符串的大小，使用字典顺序，与当前的locale有关。

（3）.关于字符串比较。[...]、[[...]]中都可以对字符串进行比较，比较的顺序是"字典顺序"。对ascii字符来讲，码表中排列在前的较小，如A<B，A<a, 1<2。再强调一次，这里只要用了"<"、">"，就表示是字符串比较，那么9 > 100为真，因为这实际上等价于‘9’ > ‘100’，9在码表中排在1后面，所以字符串"9"大于字符串"100"。只要搞清楚了何时是算术比较，何时是串比较，一般就不会出错了。

（4）建议在使用数值比较的时候，使用let，(())命令，否则容易出错；

2.1  “[“用法

$ [ 2 -lt 10 ]&&echo true&&echo false

true

$  [ 2 -gt 10 ]&&echo true||echo false

false

$  [ 2 \< 10 ]&&echo true||echo false  #you should use "\<"

false

$  [ 2 \> 10 ]&&echo true||echo false  #you should use "\>"

true

2.2     “[[“用法

$  [[ 2 -gt 10 ]]&&echo true||echo false

false

$  [[ 2 -lt 10 ]]&&echo true||echo false

true

$  [[ 2 < 10 ]]&&echo true||echo false

false

$  [[ 2 > 10 ]]&&echo true||echo false

true

 

3.        相同：都支持简单的模式匹配

这里的模式匹配要简单得多，类似文件名的统配符的扩展规则。还要注意等号右端的模式不能用引号括起，使用引用关闭了某些元字符的特殊功能

 

3.1  “[“用法

$ [ test = test ]&&echo true||echo false  #normal compare

true

$ [ test = t*t ]&&echo true||echo false  #pattern match.

true

$ [ test = t..t ]&&echo true||echo false  #not match.

false

$ [ test = t??t ]&&echo true||echo false  #note that "?", not "." stands for one single character here

true

$ [ test = "t??t" ]&&echo true||echo false #alert: don't quote the pattern，使用引用关闭了?的特殊功能

false

 

3.2  “[[“用法

$ [[ test = test ]]&&echo true||echo false  #normal compare

true

$ [[ test = t*t ]]&&echo true||echo false  #pattern match.

true

$ [[ test = t..t ]]&&echo true||echo false  #not match.

false

$ [[ test = t??t ]]&&echo true||echo false  #note that "?", not "." stands for one single character here

true

$ [[ test = "t??t" ]]&&echo true||echo false # alert: don't quote the pattern，使用引用关闭了?的特殊功能

false

 

4.        不同点

4.1     逻辑与和逻辑或

（1）"["：逻辑与："-a"；逻辑或："-o"；

（2）"[["：逻辑与："&&"；逻辑或："||"

$ [[ 1 < 2 && b > a ]]&&echo true||echo false

true

$ [[ 1 < 2 -a b > a ]]&&echo true||echo false

bash: syntax error in conditional expression

bash: syntax error near `-a'

$ [ 1 < 2 -a b > a ]&&echo true||echo false

true

$ [ 1 < 2 && b > a ]&&echo true||echo false  #wrong syntax

bash: [: missing `]'

false

$ [ 1 < 2 \&\& b > a ]&&echo true||echo false  #aslo wrong

bash: [: &&: binary operator expected

false

 

4.2     命令行参数

（1）[ ... ]为shell命令，所以在其中的表达式应是它的命令行参数，所以串比较操作符">" 与"<"必须转义，否则就变成IO重定向了；

（2）由于"[["是关键字，不会做命令行扩展，所以在[[中"<"与">"不需转义，但是相对的语法就稍严格些。例如在[ ... ]中可以用引号括起操作符，因为在做命令行扩展时会去掉这些引号，而在[[ ... ]]则不允许这样做；

$ [ "-z" "" ]&&echo true||echo false

true

$ [ -z "" ]&&echo true||echo false

true

$ [[ "-z" "" ]]&&echo true||echo false

bash: conditional binary operator expected

bash: syntax error near `""'

$ [[ -z "" ]]&&echo true||echo false

true

 

4.3  [[ ... ]]进行算术扩展，而[ ... ]不做

$ [[ 99+1 -eq 100 ]]&&echo true||echo false

true

$ [ 99+1 -eq 100 ]&&echo true||echo false

bash: [: 99+1: integer expression expected

false

$ [ $((99+1)) -eq 100 ]&&echo true||echo false

true

 

4.4  正则表达式匹配"=~"

regular expression match. This operator was introduced with version 3 of Bash.
The =~ Regular Expression matching operator within a double brackets test expression.

# REF
[Shell中[和[[的异同](https://blog.csdn.net/ysdaniel/article/details/7905818)