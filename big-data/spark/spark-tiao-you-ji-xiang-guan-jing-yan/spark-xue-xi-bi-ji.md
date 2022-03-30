# Spark学习笔记



## 前言&#x20;

Spark 作为目前最火的技术栈~~或许 大概 应该 maybe 没有之一了吧~~，看上去很厉害，实际上也很厉害。。。 ~~去年，有个东西还准备自己造轮子解决，后来耽搁了，上周一搜，已经有大神造好了轮子~~ 本篇，作为自己学习Spark 的一个记录，不涉及Spark 的具体介绍，主要是一些学习思路和学习资料的整理，资源都来自网络及社区，侵删。

## 前置条件

### 编程语言

学习使用Spark，需要有一定的基础知识，在编程语言的方面，目前支持了Scala、Java、Python、R。个人建议是Scala 或 Java。

*   Scala Scala 作为Spark 的开发语言，简洁、优雅、语法丰富、支持lamba表达式，能够明显的提高开发效率，唯一的问题就是代码的可读性稍差~~刚开始用Scala 写Spark 的时候，都用笔在纸上写出各个RDD之间的转化~~ 。此外，会java的程序员一抓一大把，会Scala的略少，可能有一个学习的过程。

    **快学Scala** 很不错的书 ~~恩，买了到现在还跟新的一样~~ PDF [下载链接](https://pan.baidu.com/s/1dEQiSEX) 密码：l9ef，请支持正版

    **为Java程序员编写的Scala的入门教程** 非常实用的教程~~1个小时从入门到精通~~ 该教程仅在对语言和其编译器进行简要介绍。目的读者是那些已经具有一定编程经验，而想尝试一下Scala语言的人们。要阅读该教程，应当具有基础的面向对象编程的概念，尤其是Java语言的。 [传送门](https://www.iteblog.com/archives/1325.html)

    **Scala语言规范** Scala语言定义和一些核心库模块的参考手册，可以当工具书查。 [传送门](http://www.scala-lang.org/docu/files/Scala%E8%AF%AD%E8%A8%80%E8%A7%84%E8%8C%83.pdf)

    **菜鸟教程** http://www.runoob.com/scala/scala-tutorial.html
* Java ~~这个没啥好说的~~ 真是初学者的话，网上随便搜个[教程](http://www.runoob.com/java/java-tutorial.html)，装好JDK和IDE，先写个hello world，再了解下面向对象的知识，然后看看多线程，差不多可以边看API边用着了。剩下就是左手[Google](https://www.baidu.com) 右手[Stack Overflow](https://stackoverflow.com)了。 参考书的话，推荐[Java 核心技术](https://pan.baidu.com/s/1eSIpBpC) 和 [Thinking In Java](https://pan.baidu.com/s/1i5BxQoH) ~~这么经典的书，你不买套正版的，好意思说自己是程序员么~~
*   Python 胶水型语言，即用即贴。各种神奇的库，机器学习、数据分析方便的一比。工业用就差那么点了。

    **廖雪峰的python教程** 廖雪峰老师的这个[教程](https://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000)作为入门足够了。剩下的，看着[Spark 的官方API](http://spark.apache.org/docs/latest/api/python/index.html)，自己搜吧。
* R 如果你本身就是搞数据分析出身，用R习惯了，那也不用学什么了。 如果你本身不会R，上面三个还不够你学么！！！

### 算法知识

* 机器学习 如果你要使用 Spark MLlib 的话，你需要一定的机器学习基础，至少得知道你用的算法是什么吧。 周志华老师的[机器学习](https://pan.baidu.com/s/1mhG1oeO) 斯坦福大学公开课 [机器学习课程](http://open.163.com/special/opencourse/machinelearning.html) Mitchell 的[机器学习](https://pan.baidu.com/s/1dF94n4H)
* 图论 如果要使用Graph X 的话，需要一些图论 和 图计算的基本知识。 [图论及其应用](https://pan.baidu.com/s/1dEFtAz3)

### SQL 使用Spark SQL 的话，你需要一些SQL 的基本知识。

## 入门教程&#x20;

spark 的教程方法很多，最实用的是去官网，看Quick Start。 其他的，网上也有很多，整理了几个个人觉得很棒的教程。

* Spark 入门实战 [Spark 入门实战系列](http://blog.csdn.net/yirenboy/article/details/47291765) 这套教程从Spark 的生态圈介绍开始，涉及了基础的开发环境搭建，基本概念的介绍，以及Spark 组件的使用。理论结合实践，demo 代码（基于Scala）一应俱全。可谓入门的不二之选。
* Spark快速大数据分析 [Spark快速大数据分析](http://www.ituring.com.cn/book/1558)，这本书200页左右，介绍了Spark RDD 的基本操作，以及其余模块的基本使用，非常适合初学者。
* IDE推荐 **首选**是IDEA **次选**是Eclipse ~~当然，你可以用vim，没人拦着你~~ **此外**还有一个方式，在已经搭建好Spark 环境的集群上，直接进入[Spark shell](http://blog.csdn.net/yeruby/article/details/41043039)，都不用编译工程，可谓方便快捷，在实验一些逻辑的时候，能提高不少效率 **还有**Spark 的Master 设置为local 的时候，可以直接运行IDE看到结果，无需打包上传至服务器，使用spark-submit 的方式提交。验证代码逻辑可用。

## 进阶教程

* Spark 高级大数据分析 [Spark 高级大数据分析](http://www.ituring.com.cn/book/1668)介绍了如何利用Spark进行大规模数据分析的若干模式，通过实例向讲述了怎样解决分析型问题。
* Apache Spark 源码剖析 [Apache Spark 源码剖析](https://www.amazon.cn/Apache-Spark%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90-%E8%AE%B8%E9%B9%8F/dp/B00U0A9L3C/ref=sr\_1\_6?ie=UTF8\&qid=1426825361\&sr=8-6\&keywords=spark) 以Spark 1.02版本源码为切入点，着力于探寻Spark所要解决的主要问题及其解决办法，通过一系列精心设计的小实验来分析每一步背后的处理逻辑。
* Spark 源码走读 该[系列](http://www.cnblogs.com/hseagle/p/3664933.html) 从Spark Job 的提交开始，进行Spark的源码走读。 ~~恩 他就是上面那本书的作者~~
* Spark SQL 源码分析 盛利大神 源码阅读的[笔记](http://blog.csdn.net/oopsoom/article/details/38257749)
* Spark Streaming 源码解析系列 来自 腾讯 广点通 技术团队出品的[源码解析](https://github.com/lw-lin/CoolplaySpark/tree/master/Spark%20Streaming%20%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90%E7%B3%BB%E5%88%97)
* Spark MLlib 源码阅读 [这个](http://blog.csdn.net/column/details/14894.html) 按照MLlib 中的次序，进行介绍。
* RDD paper Spark 诞生地 UCB 的论文[Resilient Distributed Datasets: A Fault-Tolerant Abstraction for In-Memory Cluster Computing](http://www.ece.eng.wayne.edu/\~sjiang/ECE7650-winter-16/12-spark-questions.pdf) 系统的阐述了RDD的设计初衷和基本架构，对深入理解Spark 有很大的帮助。 这是一篇[中文翻译](http://blog.sciencenet.cn/blog-425672-520947.html)
* API 在开发的过程中，没有什么比官方文档更好的老师了。自己在[官网](http://spark.apache.org/docs/latest/index.html)找对应版本的API即可。
* 源码 在github 上有Spark 的源码，个人感觉目前用的比较多的是1.6.3 和 2.1.0 两个版本。Spark 1.6 和 2.x 设计上确有一些地方有了改动，但是总体的实现思路，还是类似的，看源码的话，根据自己当下使用的版本，开始看吧。一些设计理念可以借助上文的源码走读加深理解。

## 一些站点

* [CSDN 专栏](http://spark.csdn.net)
* [过往记忆-Spark专栏](https://www.iteblog.com/archives/category/spark/) ~~依稀记得，当时年少，懵懂无知，网还被墙，一遇问题，只能度娘，结果搜到的解决方法，大多来自过往记忆~~
* [邵塞塞](http://jerryshao.me) 早期Spark contributor之一
* [Matei Zaharia](https://amplab.cs.berkeley.edu/author/mzaharia) UCB 的大神
* [databricks](https://databricks.com/blog)
