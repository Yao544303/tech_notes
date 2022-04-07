# find命令参数详解
find一些常用参数的一些常用实例和一些具体用法和注意事项。

1. 使用name 选项
文件名选项是find命令最常用的选项，要么单独使用该选项，要么和其他选项一起使用。  可以使用某种文件名模式来匹配文件，记住要用引号将文件名模式引起来。  不管当前路径是什么，如果想要在自己的根目录$HOME中查找文件名符合*.log的文件，使用~作为 'pathname'参数，波浪号~代表了你的$HOME目录。
find ~ -name "*.log" -print  
想要在当前目录及子目录中查找所有的‘ *.log‘文件，可以用： 
find . -name "*.log" -print  
想要的当前目录及子目录中查找文件名以一个大写字母开头的文件，可以用：  
find . -name "[A-Z]*" -print  
想要在/etc目录中查找文件名以host开头的文件，可以用：  
find /etc -name "host*" -print  
想要查找$HOME目录中的文件，可以用：  
find ~ -name "*" -print 或find . -print  
要想让系统高负荷运行，就从根目录开始查找所有的文件。  
find / -name "*" -print  
如果想在当前目录查找文件名以一个个小写字母开头，最后是4到9加上.log结束的文件：  
命令：
find . -name "[a-z]*[4-9].log" -print

2. 用perm选项
按照文件权限模式用-perm选项,按文件权限模式来查找文件的话。最好使用八进制的权限表示法。  
如在当前目录下查找文件权限位为755的文件，即文件属主可以读、写、执行，其他用户可以读、执行的文件，可以用： 
find . -perm 755 -print

3. 忽略某个目录：
如果在查找文件时希望忽略某个目录，因为你知道那个目录中没有你所要查找的文件，那么可以使用-prune选项来指出需要忽略的目录。在使用-prune选项时要当心，因为如果你同时使用了-depth选项，那么-prune选项就会被find命令忽略。如果希望在test目录下查找文件，但不希望在test/test3目录下查找，可以用：  
命令：
find test -path "test/test3" -prune -o -print

4. 使用find查找文件的时候怎么避开某个文件目录： 
实例1：在test 目录下查找不在test4子目录之内的所有文件
命令：
find test -path "test/test4" -prune -o -print

5. 使用user和nouser选项
实例1：在$HOME目录中查找文件属主为peida的文件 
命令：
find ~ -user peida -print  
实例2：在/etc目录下查找文件属主为peida的文件: 
命令：
find /etc -user peida -print  
说明：
实例3：为了查找属主帐户已经被删除的文件，可以使用-nouser选项。在/home目录下查找所有的这类文件
命令：
find /home -nouser -print
说明：
这样就能够找到那些属主在/etc/passwd文件中没有有效帐户的文件。在使用-nouser选项时，不必给出用户名； find命令能够为你完成相应的工作。

