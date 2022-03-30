# PyCharm\_Tips

## pycharm pep8 代码格式化

* 设置每行最长字符数为 79

```
File-Settings-Editor-Code Style-Right Margin
```

* 使用ctrl+alt+L 快捷键格式化代码
* 运行tox命令检查，解决其他剩余问题

```
PS：pycharm默认就监测pep8代码格式，只是提示是白色warn，隐藏在各种warn信息中，不
明显，大家可以把pep8的提示级别修改为 error：
File-Settings-Editor-Inspection
这样不符合pep8的代码会直接报红，可以在开发中直接识别修改
```

## 配置pycharm + spark开发环境



1. 安装python （anaconda）
2. 安装pycharm
3. 安装spark （下载一个tar包，解压到对应目录即可
4. 配置python spark 的环境变量

```
PYTHON2 C:\dev\Anaconda3\envs\py2
PYTHON3 C:\dev\Anaconda3
SPARK_HOME C:\dev\spark1.6.3
PYTHONPATH %SPARK_HOME%\Python;%SPARK_HOME%\python\lib\py4j-0.9-src.zip
我的环境变量有点多=。=！！ 然后 根据你使用的python ，将python 的环境变量加入path
```

5\. pycharm 中设置，选择python 的path
