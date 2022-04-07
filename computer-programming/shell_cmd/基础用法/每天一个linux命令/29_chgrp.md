# chgrp 命令
在lunix系统里，文件或目录的权限的掌控以拥有者及所诉群组来管理。可以使用chgrp指令取变更文件与目录所属群组，这种方式采用群组名称或群组识别码都可以。Chgrp命令就是change group的缩写！要被改变的组名必须要在/etc/group文件内存在才行。

## 命令格式
chgrp [option] [group] file

## 命令功能
chgrp命令可采用群组名称或群组识别码的方式改变文件或目录的所属群组。使用权限是超级用户。 

## 命令参数
必要参数:
-c 当发生改变时输出调试信息
-f 不显示错误信息
-R 处理指定目录以及其子目录下的所有文件
-v 运行时显示详细的处理信息
--dereference 作用于符号链接的指向，而不是符号链接本身
--no-dereference 作用于符号链接本身
选择参数:
--reference=<文件或者目录>
--help 显示帮助信息
--version 显示版本信息

## 使用实例
1. 改变文件的群组属性
chgrp -v bin log

2. 根据指定文件改变文件的群组属性
chgrp --reference=log2012.log log2013.log

3. 改变指定目录以及其子目录下的所有文件的群组属性 
chgrp -R bin test6

4. 通过群组识别码改变文件群组属性
chgrp -R 100 test6
通过群组识别码改变文件群组属性，100为users群组的识别码，具体群组和群组识别码可以去/etc/group文件中查看
