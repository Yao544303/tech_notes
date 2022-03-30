# IDEA\_Tips

## 新开发环境基本配置

* 快捷键调整
* 主题调节
* 编码规范和模板

## intellij 出现“Usage of API documented as @since 1.6+”的解决办法

```
Usage of API documented as @since 1.6+ This inspection finds all usages of
methods that have @since tag in their documentation. This may be useful when
development is performed under newer SDK version as the target platform for
production
File ->Project Structure->Project Settings -> Modules -> 你的Module名字 ->
Sources -> Language Level->选个默认的就行。
```

## 无法创建package 和 class

将普通文件夹设为Sources Root

[Idea maven项目不能新建package和class的解决](http://blog.csdn.net/qq\_24949727/article/details/52097838)

## Intellij IDEA 14.1.4 Scala开发环境搭建

[Intellij IDEA 14.1.4 Scala开发环境搭建](http://blog.csdn.net/lovehuangjiaju/article/details/47778671)

## IDEA源码阅读技巧

1. 最近访问的: Ctrl + E
2. 生成类图: Ctrl+Alt+Shift+U
3. 类里面定义的变量在那些地方被调用，那么需要掌握一下: Ctrl+Alt+F7
4. 看一个类里面有那些方法: Alt+7 或者 Ctrl+F12
5. 看一个类/方法的实现类/方法: Ctrl+Alt+B
6. 某个方法的调用链关系: Ctrl+Alt+H
7. 查看某个方法被那些地方调用:Ctrl+B 或 Ctrl + Click
8. 计算表达式 :Alt+F8

[使用IDEA 阅读源码的技巧](https://yq.aliyun.com/articles/666136)

## IDEA内存优化

编辑如下文件

```
\IntelliJ IDEA 9\bin\idea.exe.vmoptions
-----------------------------------------
-Xms64m
-Xmx256m
-XX:MaxPermSize=92m
-ea
-server
-Dsun.awt.keepWorkingSetOnMinimize=true
```

## SVN 管理

把SVN库添加到IDEA中 SETTING -> VERSION CONTROL -> VCS = SVBVERSION

## 重要的设置

* 不编译某个MODULES的方法，但在视图上还是有显示 :SETTINGS -> COMPILER -> EXCLUDES&#x20;
* 不编译某个MODULES，并且不显示在视图上: MODULES SETTINGS -> (选择你的MODULE) -> SOURCES -> EXCLUDED -> 整个工程文件夹

## IDEA编码设置3步曲

1. FILE -> SETTINGS -> FILE ENCODINGS -> IDE ENCODING
2. FILE -> SETTINGS -> FILE ENCODINGS -> DEFAULT ENCODING FOR PROPERTIES FILES
3. FILE -> SETTINGS -> COMPILER -> JAVA COMPILER -> ADDITIONAL COMMAND LINE PARAMETERS加上参数 -ENCODING UTF-8 编译GROOVY文件的时候如果不加，STRING S = "中文"; 这样的GROOVY文件编译不过去.

## 编译中添加其他类型文件比如 \*.TXT \*.INI

FILE -> SETTINGS -> RESOURCE PATTERNS

## 改变编辑文本字体大小

FILE -> SETTINGS -> EDITOR COLORS & FONTS -> FONT -> SIZE

## 修改智能提示快捷键

FILE -> SETTINGS -> KEYMAP -> MAIN MENU -> CODE -> COMPLETE CODE -> BASIC

## 显示文件过滤

FILE -> SETTINGS -> FILE TYPES -> IGNORE FILES...&#x20;

下边是我过滤的类型,区分大小写的 CVS;SCCS;RCS;rcs;.DS\_Store;.svn;.pyc;.pyo;_.pyc;_.pyo;.git;_.hprof;\_svn;.sbas;.IJI._;vssver.scc;vssver2.scc;._;_.iml;_.ipr;_.iws;\*.ids

## 在PROJECT窗口中快速定位,编辑窗口中的文件

在编辑的所选文件按ALT+F1, 然后选择PROJECT VIEW

## intellij idea 自动生成test单元测试

在要测试的类上按快捷键ctrl + shift + t，选择Create New Test，在出现的对话框的下面member内勾选要测试的方法，点击ok

或者点击菜单栏Navigate–》test，选择Create New Test，在出现的对话框的下面member内勾选要测试的方法，点击ok

## IntelliJ Idea 依赖包下载成功，代码里无法import问题解决方法

1. 今天clone一个github上的基于maven的项目IntelliJ Idea 依赖包下载成功，代码里无法import。解决方法：删掉原来的.iml，刷新。
2. 之前编码一直用Eclipse，现在迷恋上了IntelliJ Idea，真的挺好用的，不过也遇到了一些问题，如题，网上有的说清除缓存重启、有的说删除.iml文件...说啥的都有，按这些方法都没解决，最后查看了一下iml文件，找到了原因，文件中有这么一行

```
<orderEntry type="library" name="Maven: xxx-xxx-xxx:0.0.1-SNAPSHOT" level="project" />
xxx-xxx-xxx:0.0.1-SNAPSHOT这是自己写的另外一个项目，这样写就不行，改成下面的写法就可以了
<orderEntry type="module" module-name="xxx-xxx-xxx" />
```

## REF

* [IDEA 基本配置](http://blog.csdn.net/frankcheng5143/article/details/50779149)
* [IDE神器intellij idea的基本使用](https://www.cnblogs.com/newpanderking/p/4887981.html)
* [IDEA 常用快捷键](https://www.cnblogs.com/zhangpengshou/p/5366413.html)
* [使用Eclipse风格快捷键](http://blog.csdn.net/u010180031/article/details/51030776)
* [导入Eclipse 注释](https://blog.jetbrains.com/idea/2014/01/intellij-idea-13-importing-code-formatter-settings-from-eclipse/)
