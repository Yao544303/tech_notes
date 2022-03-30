# Spark开发环境搭建

## Spark本地安装

* Java 安装
* Spark 安装
* PySpark 安装

### Java安装

这一部分不多赘述，配置好Java 环境变量即可。

### Spark 安装

在[官网](http://spark.apache.org/downloads.html)下载所需版本的Spark 压缩包

![](http://upload-images.jianshu.io/upload\_images/3698622-0b4a4a91b763512a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

解压至对应目录，如 C:\dev\spark1.6.3 配置环境变量 ![](http://upload-images.jianshu.io/upload\_images/3698622-fe96697c1ec0e859.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这时，进入cmd 命令行，可以启动。 ![](http://upload-images.jianshu.io/upload\_images/3698622-9b047f516b942438.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### Pyspark 安装

要求在本机已经安装好Spark。此外python 3.6 版本不兼容Spark 1.6，使用时需要注意。 新增环境变量：PYTHONPATH 值为：%SPARK\_HOME%\Python;%SPARK\_HOME%\python\lib\py4j-0.9-src.zip

同时，在python 的配置的Lib\site-packages 中新增pyspark.pth 文件，内容为

```
C:\dev\spark1.6.3\python
```

MAC 为，在对应环境的 Lib\site-packages 下 新增pyspark.pth 文件

重启CMD ，输入pyspark 即可 ![](http://upload-images.jianshu.io/upload\_images/3698622-791496f526c654d9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

ubuntu 下搭建 参见 [这篇说明](http://blog.csdn.net/u013475704/article/details/78647552)

## 开发环境搭建

### Scala

搭建一个maven 工程即可pom.xml 如下：

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.ych</groupId>
  <artifactId>ychTestSpark4S</artifactId>
  <version>1.0-SNAPSHOT</version>
  <inceptionYear>2008</inceptionYear>
    <properties>
        <spark.version>1.6.2</spark.version>
        <scala.version>2.10</scala.version>
    </properties>

  <repositories>
    <repository>
      <id>scala-tools.org</id>
      <name>Scala-Tools Maven2 Repository</name>
      <url>http://scala-tools.org/repo-releases</url>
    </repository>
  </repositories>

  <pluginRepositories>
    <pluginRepository>
      <id>scala-tools.org</id>
      <name>Scala-Tools Maven2 Repository</name>
      <url>http://scala-tools.org/repo-releases</url>
    </pluginRepository>
  </pluginRepositories>



  <dependencies>
    <dependency>
      <groupId>org.apache.spark</groupId>
      <artifactId>spark-core_${scala.version}</artifactId>
      <version>${spark.version}</version>
    </dependency>
    <dependency>
      <groupId>org.apache.spark</groupId>
      <artifactId>spark-streaming_${scala.version}</artifactId>
      <version>${spark.version}</version>
    </dependency>
    <dependency>
      <groupId>org.apache.spark</groupId>
      <artifactId>spark-sql_${scala.version}</artifactId>
      <version>${spark.version}</version>
    </dependency>
    <dependency>
      <groupId>org.apache.spark</groupId>
      <artifactId>spark-hive_${scala.version}</artifactId>
      <version>${spark.version}</version>
    </dependency>
    <dependency>
      <groupId>org.apache.spark</groupId>
      <artifactId>spark-mllib_${scala.version}</artifactId>
      <version>${spark.version}</version>
    </dependency>

    <dependency>
      <groupId>org.apache.avro</groupId>
      <artifactId>avro</artifactId>
      <version>1.7.7</version>
    </dependency>


    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.4</version>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>org.specs</groupId>
      <artifactId>specs</artifactId>
      <version>1.2.5</version>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>com.databricks</groupId>
      <artifactId>spark-csv_2.10</artifactId>
      <version>1.0.3</version>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <plugin>
        <groupId>org.scala-tools</groupId>
        <artifactId>maven-scala-plugin</artifactId>
        <executions>
          <execution>
            <goals>
              <goal>compile</goal>
              <goal>testCompile</goal>
            </goals>
          </execution>
        </executions>
        <configuration>
          <scalaVersion>${scala.version}.6</scalaVersion>
          <args>
            <arg>-target:jvm-1.5</arg>
          </args>
        </configuration>
      </plugin>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-eclipse-plugin</artifactId>
        <configuration>
          <downloadSources>true</downloadSources>
          <buildcommands>
            <buildcommand>ch.epfl.lamp.sdt.core.scalabuilder</buildcommand>
          </buildcommands>
          <additionalProjectnatures>
            <projectnature>ch.epfl.lamp.sdt.core.scalanature</projectnature>
          </additionalProjectnatures>
          <classpathContainers>
            <classpathContainer>org.eclipse.jdt.launching.JRE_CONTAINER</classpathContainer>
            <classpathContainer>ch.epfl.lamp.sdt.launching.SCALA_CONTAINER</classpathContainer>
          </classpathContainers>
        </configuration>
      </plugin>
    </plugins>
  </build>
  <reporting>
    <plugins>
      <plugin>
        <groupId>org.scala-tools</groupId>
        <artifactId>maven-scala-plugin</artifactId>
        <configuration>
          <scalaVersion>${scala.version}</scalaVersion>
        </configuration>
      </plugin>
    </plugins>
  </reporting>
</project>
```

### Java 开发环境

同Scala

### python

设定好，需要使用的python 环境即可。 spyder 根据anaconda 设定的python 环境，选择对应的spyder 启动即可。 pycharm 如下配置： ![](http://upload-images.jianshu.io/upload\_images/3698622-8b4c2e09d180aee1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
