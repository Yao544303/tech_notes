# SparkSubmit 源码走读1\_从提交脚本到类

## 整体流程参考这个图

![image](https://img-blog.csdn.net/20170712134317873?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZG9feW91cnNlbGZfZ29fb24=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## 整个流程，涉及3个sh，以两个类作为入口

### spark-submit 部分

```
spark-submit 
-----> spark-class 
    -----> load-spark-env.sh 加载conf 中的spark-env.sh 中的环境变量，配置scala 的版本
-----> 返回spark-class
    -----> 一系列环境变量的校验，依赖包目录的校验、设置
    -----> 执行 org.apache.spark.launcher.Main
    -----> 执行 org.apache.spark.deploy.SparkSubmit
```

#### 执行 org.apache.spark.launcher.Main

```
org.apache.spark.launcher.Main
----->调用 SparkSubmitCommandBuilder 构建方法，根据输入参数列表构造builder
    -----> 调用Parser 通过正则匹配，解析参数列表中的option 和 args（同事校验，格式有问题就报错）
----->调用 SparkSubmitCommandBuilder.buildCommand 方法构造命令，将map 处理为list<String>
----->调用 prepareBashCommand (windows 有另外的方法) 生成启动命令，并print,print 的结果作为 org.apache.spark.deploy.SparkSubmit 的输入
```

#### 执行 org.apache.spark.deploy.SparkSubmit

```
org.apache.spark.deploy.SparkSubmit
-----> 根据入参，构造SparkSubmitArguments 主要工作：解析命令行参数，合并属性文件中的配置参数，载入默认环境变量参数
    -----> SparkSubmitArgumentsParser
    -----> SparkSubmitOptionParser 和 Main 中的 Parser 类似，可以理解为，Main 进行校验，拼接出来符合要求的参数，然后SparkSubmit 接收。
        -----> 合并默认的Spark参数 mergeDefaultSparkProperties，凡是--conf 中没有的默认项，都将默认参数引入
        -----> 忽略不符合条件的conf 中的入参，ignoreNonSparkProperties ，凡是--conf 中不是以"spark." 开头的参数，去除
        -----> 载入环境参数 loadEnvironmentArguments ，即master 等非--conf 参数，假如命令行设置了，以命令行的为准，否则用默认参数 （这里可以看出，命令行的优先级>默认项，结合spark-class 中spark-env.sh 的操作，环境变量参数的优先级就是，命令行>spark-env.sh(conf)> 默认配置），这里会设置action参数，默认submit。
        -----> 调用 validateSubmitArgument 校验，不同的action有不同的验证方式。
-----> submit
    -----> prepareSubmitEnviroment 设置应用程序部署方式，设置应用程序主类名，
    -----> doRunMain
        -----> proxyUser 权限校验
        -----> runMain 设置MainClass 参数等，最后通过mainMethod.invoke(null, childArgs.toArray) 通过反射机制，找到需要执行的类，执行
```

## sh部分

1. spark-submit.sh

```
# 判断SPARK_HOME 这个环境变量是否存在，不存在，则以提交脚本的上级目录作为SPARK_HOME的值
if [ -z "${SPARK_HOME}" ]; then
  export SPARK_HOME="$(cd "`dirname "$0"`"/..; pwd)"
fi

# disable randomized hash for string in Python 3.3+
export PYTHONHASHSEED=0

# 执行spark-class 脚本
exec "${SPARK_HOME}"/bin/spark-class org.apache.spark.deploy.SparkSubmit "$@"
```

1. spark-class

```
if [ -z "${SPARK_HOME}" ]; then
  export SPARK_HOME="$(cd "`dirname "$0"`"/..; pwd)"
fi

# 执行 load-spark-env.sh 脚本 加载conf 配置文件目录
. "${SPARK_HOME}"/bin/load-spark-env.sh

# Find the java binary
# 判断JAVA_HOME 是否设置
if [ -n "${JAVA_HOME}" ]; then
  RUNNER="${JAVA_HOME}/bin/java"
else
  if [ `command -v java` ]; then
    RUNNER="java"
  else
    echo "JAVA_HOME is not set" >&2
    exit 1
  fi
fi

# Find assembly jar
# 判断spark 依赖包目录是否存在
SPARK_ASSEMBLY_JAR=
if [ -f "${SPARK_HOME}/RELEASE" ]; then
  ASSEMBLY_DIR="${SPARK_HOME}/lib"
else
  ASSEMBLY_DIR="${SPARK_HOME}/assembly/target/scala-$SPARK_SCALA_VERSION"
fi

# 判断依赖包是否存在？ 是否存在重复依赖？
GREP_OPTIONS=
num_jars="$(ls -1 "$ASSEMBLY_DIR" | grep "^spark-assembly.*hadoop.*\.jar$" | wc -l)"
if [ "$num_jars" -eq "0" -a -z "$SPARK_ASSEMBLY_JAR" -a "$SPARK_PREPEND_CLASSES" != "1" ]; then
  echo "Failed to find Spark assembly in $ASSEMBLY_DIR." 1>&2
  echo "You need to build Spark before running this program." 1>&2
  exit 1
fi
if [ -d "$ASSEMBLY_DIR" ]; then
  ASSEMBLY_JARS="$(ls -1 "$ASSEMBLY_DIR" | grep "^spark-assembly.*hadoop.*\.jar$" || true)"
  if [ "$num_jars" -gt "1" ]; then
    echo "Found multiple Spark assembly jars in $ASSEMBLY_DIR:" 1>&2
    echo "$ASSEMBLY_JARS" 1>&2
    echo "Please remove all but one jar." 1>&2
    exit 1
  fi
fi

# 设置依赖包目录
SPARK_ASSEMBLY_JAR="${ASSEMBLY_DIR}/${ASSEMBLY_JARS}"

# 设置LAUNCH_CLASSPATH 用于后面的入参
LAUNCH_CLASSPATH="$SPARK_ASSEMBLY_JAR"

# Add the launcher build dir to the classpath if requested.
if [ -n "$SPARK_PREPEND_CLASSES" ]; then
  LAUNCH_CLASSPATH="${SPARK_HOME}/launcher/target/scala-$SPARK_SCALA_VERSION/classes:$LAUNCH_CLASSPATH"
fi

export _SPARK_ASSEMBLY="$SPARK_ASSEMBLY_JAR"

# For tests
if [[ -n "$SPARK_TESTING" ]]; then
  unset YARN_CONF_DIR
  unset HADOOP_CONF_DIR
fi

# The launcher library will print arguments separated by a NULL character, to allow arguments with
# characters that would be otherwise interpreted by the shell. Read that in a while loop, populating
# an array that will be used to exec the final command.
# 执行org.apache.spark.launcher.Main作为Spark应用程序的主入口
# 注意，这里org.apache.spark.launcher.Main 位于launch 目录下，不是core目录下
CMD=()
while IFS= read -d '' -r ARG; do
  CMD+=("$ARG")
done < <("$RUNNER" -cp "$LAUNCH_CLASSPATH" org.apache.spark.launcher.Main "$@")
exec "${CMD[@]}"
```

1. load-spark-env.sh

```
if [ -z "${SPARK_HOME}" ]; then
  export SPARK_HOME="$(cd "`dirname "$0"`"/..; pwd)"
fi

# 当SPARK_ENV_LOADED 不存在时，将其设为1
if [ -z "$SPARK_ENV_LOADED" ]; then
  export SPARK_ENV_LOADED=1

  # Returns the parent of the directory this script lives in.
  parent_dir="${SPARK_HOME}"

  # 加载conf 配置文件目录
  user_conf_dir="${SPARK_CONF_DIR:-"$parent_dir"/conf}"
  
  # 如果conf目录下的spark-env.sh 存在，则执行
  # spark-env.sh 中主要配置一些环境变脸，如不使用默认值，有修改的，则需要执行spark-env.sh
  if [ -f "${user_conf_dir}/spark-env.sh" ]; then
    # Promote all variable declarations to environment (exported) variables
    set -a
    . "${user_conf_dir}/spark-env.sh"
    set +a
  fi
fi


# Setting SPARK_SCALA_VERSION if not already set.

if [ -z "$SPARK_SCALA_VERSION" ]; then

  ASSEMBLY_DIR2="${SPARK_HOME}/assembly/target/scala-2.11"
  ASSEMBLY_DIR1="${SPARK_HOME}/assembly/target/scala-2.10"

  if [[ -d "$ASSEMBLY_DIR2" && -d "$ASSEMBLY_DIR1" ]]; then
    echo -e "Presence of build for both scala versions(SCALA 2.10 and SCALA 2.11) detected." 1>&2
    echo -e 'Either clean one of them or, export SPARK_SCALA_VERSION=2.11 in spark-env.sh.' 1>&2
    exit 1
  fi

  if [ -d "$ASSEMBLY_DIR2" ]; then
    export SPARK_SCALA_VERSION="2.11"
  else
    export SPARK_SCALA_VERSION="2.10"
  fi
fi
```

## REF

* [浅谈Spark几种不同的任务提交相关脚本](https://blog.csdn.net/lovehuangjiaju/article/details/48768371)
* [Spark学习笔记之-Spark 命令及程序入口](https://blog.csdn.net/dandykang/article/details/49300037)
* [Spark-Core源码精读(4)、对Main类的补充说明](https://www.jianshu.com/p/399fb2ba79fa)
* [Spark源码解析之任务提交（spark-submit）篇](https://blog.csdn.net/do\_yourself\_go\_on/article/details/75005204)
