exit（0）：正常运行程序并退出程序；
exit（1）：非正常运行导致退出程序；
exit 0 可以告知你的程序的使用者：你的程序是正常结束的。如果 exit 非 0 值，那么你的程序的使用者通常会认为你的程序产生了一个错误。
在 shell 中调用完你的程序之后，用 echo $? 命令就可以看到你的程序的 exit 值。在 shell 脚本中，通常会根据上一个命令的 $? 值来进行一些流程控制。


# REF
[shell 中　exit0 exit1 的区别](https://blog.csdn.net/super_gnu/article/details/77099395)