# SparkMLlib 数据结构

## Spark MLlib 基础数据结构

### 1 本地向量

```
MLlib的本地向量主要分为两种，DenseVector和SparseVector，顾名思义，前者是用来保存稠密向量，后者是用来保存稀疏向量，其创建方式主要有一下三种（三种方式均创建了向量(1.0, 0.0, 2.0）：
```

```
import org.apache.spark.mllib.linalg.{Vector, Vectors}  
//创建一个稠密向量  
val dv : Vector = Vector.dense(1.0,0.0,3.0);  
//创建一个稀疏向量（第一种方式）  
val sv1: Vector = Vector.sparse(3, Array(0,2), Array(1.0,3.0));  
//创建一个稀疏向量（第二种方式）  
val sv2 : Vector = Vector.sparse(3, Seq((0,1.0),(2,3.0)))  
```

* 对于稠密向量：很直观，你要创建什么，就加入什么，其函数声明为Vector.dense(values : Array\[Double])
* 对于稀疏向量，当采用第一种方式时，3表示此向量的长度，第一个Array(0,2)表示的索引，第二个Array(1.0, 3.0)与前面的Array(0,2)是相互对应的，表示第0个位置的值为1.0，第2个位置的值为3
* 对于稀疏向量，当采用第二种方式时，3表示此向量的长度，后面的比较直观，Seq里面每一对都是(索引，值）的形式。

tips:由于scala中会默认包含scal.collection.immutalbe.Vector，所以当使用MLlib中的Vector时，需要显式的指明import路径

### 2 向量标签

向量标签和向量是一起的，简单来说，可以理解为一个向量对应的一个特殊值，这个值的具体内容可以由用户指定，比如你开发了一个算法A，这个算法对每个向量处理之后会得出一个特殊的标记值p，你就可以把p作为向量标签。同样的，更为直观的话，你可以把向量标签作为行索引，从而用多个本地向量构成一个矩阵（当然，MLlib中已经实现了多种矩阵）

```
import org.apache.spark.mllib.linag.Vectors  
import org.apache.spark.mllib.regression.LabeledPoint  
  
val pos = LabeledPoint(1.0, Vectors.dense(1.0, 0.0, 3.0))  
对于pos变量，第一个参数1.0的具体含义只有你自己知道咯，可以使行索引，可以使特殊值神马的
```

#### 从文件中直接读入一个LabeledPoint

MLlib提供了一种快捷的方法，可以让用户直接从文件中读取LabeledPoint格式的数据。规定其输入文件的格式为：

```
label index1:value1 index2:value2.....  
```

```
val test : RDD[LabeledPoint] = MLUtils.loadLibSVMFile(sc, "path")  
```

### 3 本地矩阵

既然是算数运算包，肯定少不了矩阵包，先上代码：

```
import org.apache.spark.mllib.linalg.{Matrix, Matrices}  
  
val dm : Matrix = Matrices.dense(3,2, Array(1.0,3.0,5.0,2.0,4.0,6.0)) 
```

上面的代码段创建了一个稠密矩阵：

```
1.0	2.0
3.0	4.0
5.0	6.0
```

很明显，创建的时候是将原来的矩阵按照列变成一个一维矩阵之后再初始化的。 tips:注意，我们创建的是稠密矩阵，不幸的事，MLlib中并没有提供稀疏矩阵的实现.

### 4 分布式矩阵

MLlib提供了三种分布式矩阵的实现，依据你数据的不同的特点，你可以选择不同类型的数据：

#### RowMatrix

RowMatrix矩阵只是将矩阵存储起来，要注意的是，此种矩阵不能按照行号访问。

```
import org.apache.spark.mllib.linalg.Vector  
import org.apache.spark.mllib.linalg.distributed.RowMatrix  
val rows: RDD[Vector] = ...//  
val mat: RowMatrix = new RowMatrix(rows)  
  
val m = mat.numRows()  
val n = mat.numCols()  
```

RowMatrix要从RDD\[Vector]构造，m是mat的行数，n是mat的列 Multivariate summary statistics

```
import org.apache.spark.mllib.linalg.Matrix  
import org.apache.spark.mllib.linalg.distributed.RowMatrix  
import org.apache.spark.mllib.stat.MultivariateStatisticalSummary  
val mat: RowMatrix = ..  
val summy : MultivariateStatisticalSummary = mat.computeColumnSummaryStatistics()  
println(summy.mean)//平均数  
```

#### IndexedRowMatrix

IndexedRowMatrix矩阵和RowMatrix矩阵的不同之处在于，你可以通过索引值来访问每一行。其他的，没啥区别。

#### CoordinateMatrix

当你的数据特别稀疏的时候怎么办？采用这种矩阵吧。

```
import org.apache.spark.mllib.linalg.distributed.{CoordinatedMatrix, MatrixEntry}  
  
val entries : RDD[MatrixEntry] = ..  
val mat: CoordinateMatrix = new CoordinateMatrix(entries)

CoordinateMatrix矩阵中的存储形式是（row，col，value），就是原始的最稀疏的方式，所以如果矩阵比较稠密，别用这种数据格式
```

### 5 libSVM

```
Label 1:value 2:value ….

Label：是类别的标识，比如上节train.model中提到的1 -1，你可以自己随意定，比如-10，0，15。当然，如果是回归，这是目标值，就要实事求是了。

Value：就是要训练的数据，从分类的角度来说就是特征值，数据之间用空格隔开

比如: -15 1:0.708 2:1056 3:-0.3333
```

需要注意的是，如果特征值为0，特征冒号前面的(姑且称做序号)可以不连续。如：

&#x20;`-15 1:0.708 3:-0.3333`&#x20;

表明第2个特征值为0，从编程的角度来说，这样做可以减少内存的使用，并提高做矩阵内积时的运算速度。我们平时在matlab中产生的数据都是没有序号的常规矩阵，所以为了方便最好编一个程序进行转化。
