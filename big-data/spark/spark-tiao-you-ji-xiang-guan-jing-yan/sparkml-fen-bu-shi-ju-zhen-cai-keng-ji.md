# SparkML分布式矩阵采坑记

最近在开发一版基于协同过滤算法的推荐系统，需要用到历史7天的搜索和所有历史订单数据，几十亿的数据参与运算。

## 分布式矩阵

分布式矩阵由长整型的行列索引值和双精度浮点型的元素值组成。它可以分布式地存储在一个或多个RDD上，MLlib提供了三种分布式矩阵的存储方案：行矩阵RowMatrix，索引行矩阵IndexedRowMatrix、坐标矩阵CoordinateMatrix和分块矩阵Block Matrix。它们都属于org.apache.spark.mllib.linalg.distributed包。

### 行矩阵

行矩阵RowMatrix是最基础的分布式矩阵类型。每行是一个本地向量，行索引无实际意义（即无法直接使用）。数据存储在一个由行组成的RDD中，其中每一行都使用一个本地向量来进行存储。由于行是通过本地向量来实现的，故列数（即行的维度）被限制在普通整型（integer）的范围内。在实际使用时，由于单机处理本地向量的存储和通信代价，行维度更是需要被控制在一个更小的范围之内。RowMatrix可通过一个RDD\[Vector]的实例来创建，如下代码所示 面向行，底层是一个以本地向量为数据项的RDD. 每行都是本地向量，long型为行号，所以会收到类型范围限制 ''' // 创建两个本地向量dv1 dv2 val dv1 : Vector = Vectors.dense(1.0,2.0,3.0) val dv2 : Vector = Vectors.dense(2.0,3.0,4.0) // 使用两个本地向量创建一个RDD\[Vector] val rows : RDD\[Vector] = sc.parallelize(Array(dv1,dv2)) // 通过RDD\[Vector]创建一个行矩阵 val mat : RowMatrix = new RowMatrix(rows)

```
//可以使用numRows()和numCols()方法得到行数和列数
mat.numRows()
mat.numCols()

// 通过computeColumnSummaryStatistics()方法获取统计摘要
val summary = mat.computeColumnSummaryStatistics()
summary.count
summary.max
summary.variance
summary.mean
summary.normL1
```

'''

### 索引行矩阵（IndexedRowMatrix）

索引行矩阵IndexedRowMatrix与RowMatrix相似，但它的每一行都带有一个有意义的行索引值，这个索引值可以被用来识别不同行，或是进行诸如join之类的操作。其数据存储在一个由IndexedRow组成的RDD里，即每一行都是一个带长整型索引的本地向量。 底层实现，是一个带行索引的RDD，这个RDD，每行是Long型索引和本地向量 与RowMatrix类似，IndexedRowMatrix的实例可以通过RDD\[IndexedRow]实例来创建。如下代码段所示（接上例）： ''' import org.apache.spark.mllib.linalg.distributed.{IndexedRow, IndexedRowMatrix} // 通过本地向量dv1 dv2来创建对应的IndexedRow // 在创建时可以给定行的索引值，如这里给dv1的向量赋索引值1，dv2赋索引值2 val idxr1 = IndexedRow(1,dv1) val idxr2 = IndexedRow(2,dv2)

```
// 通过IndexedRow创建RDD[IndexedRow]
val idxrows = sc.parallelize(Array(idxr1,idxr2))

// 通过RDD[IndexedRow]创建一个索引行矩阵
val idxmat: IndexedRowMatrix = new IndexedRowMatrix(idxrows)

idxmat.rows.foreach(println)
```

'''

### 坐标矩阵（Coordinate Matrix）

坐标矩阵CoordinateMatrix是一个基于矩阵项构成的RDD的分布式矩阵。每一个矩阵项MatrixEntry都是一个三元组：(i: Long, j: Long, value: Double)，其中i是行索引，j是列索引，value是该位置的值。坐标矩阵一般在矩阵的两个维度都很大，且矩阵非常稀疏的时候使用。

CoordinateMatrix实例可通过RDD\[MatrixEntry]实例来创建，其中每一个矩阵项都是一个(rowIndex, colIndex, elem)的三元组

坐标矩阵可以通过transpose()方法对矩阵进行转置操作，并可以通过自带的toIndexedRowMatrix()方法转换成索引行矩阵IndexedRowMatrix。但目前暂不支持CoordinateMatrix的其他计算操作。

''' import org.apache.spark.mllib.linalg.distributed.{CoordinateMatrix, MatrixEntry} // 创建两个矩阵项ent1和ent2，每一个矩阵项都是由索引和值构成的三元组 val ent1 = new MatrixEntry(0,1,0.5) val ent2 = new MatrixEntry(2,2,1.8)

```
// 创建RDD[MatrixEntry]
val entries : RDD[MatrixEntry] = sc.parallelize(Array(ent1,ent2))

// 通过RDD[MatrixEntry]创建一个坐标矩阵
val coordMat: CoordinateMatrix = new CoordinateMatrix(entries)

coordMat.entries.foreach(println)

// 将coordMat进行转置
val transMat: CoordinateMatrix = coordMat.transpose()
transMat.entries.foreach(println)

// 将坐标矩阵转换成一个索引行矩阵
val indexedRowMatrix = transMat.toIndexedRowMatrix()
 indexedRowMatrix.rows.foreach(println)
```

'''

### 分块矩阵（Block Matrix）

分块矩阵是基于矩阵块MatrixBlock构成的RDD的分布式矩阵，其中每一个矩阵块MatrixBlock都是一个元组((Int, Int), Matrix)，其中(Int, Int)是块的索引，而Matrix则是在对应位置的子矩阵（sub-matrix），其尺寸由rowsPerBlock和colsPerBlock决定，默认值均为1024。分块矩阵支持和另一个分块矩阵进行加法操作和乘法操作，并提供了一个支持方法validate()来确认分块矩阵是否创建成功。

分块矩阵可由索引行矩阵IndexedRowMatrix或坐标矩阵CoordinateMatrix调用toBlockMatrix()方法来进行转换，该方法将矩阵划分成尺寸默认为1024×1024的分块，可以在调用toBlockMatrix(rowsPerBlock, colsPerBlock)方法时传入参数来调整分块的尺寸。 下面以矩阵A（如图）为例，先利用矩阵项MatrixEntry将其构造成坐标矩阵，再转化成如图所示的4个分块矩阵，最后对矩阵A与其转置进行乘法运算： ''' import org.apache.spark.mllib.linalg.distributed.{CoordinateMatrix, MatrixEntry} import org.apache.spark.mllib.linalg.distributed.BlockMatrix

```
// 创建8个矩阵项，每一个矩阵项都是由索引和值构成的三元组
val ent1 = new MatrixEntry(0,0,1)
val ent2 = new MatrixEntry(1,1,1)
val ent3 = new MatrixEntry(2,0,-1)
val ent4 = new MatrixEntry(2,1,2)
val ent5 = new MatrixEntry(2,2,1)
val ent6 = new MatrixEntry(3,0,1)
val ent7 = new MatrixEntry(3,1,1)
val ent8 = new MatrixEntry(3,3,1)

// 创建RDD[MatrixEntry]
val entries : RDD[MatrixEntry] = sc.parallelize(Array(ent1,ent2,ent3,ent4,ent5,ent6,ent7,ent8))

// 通过RDD[MatrixEntry]创建一个坐标矩阵
val coordMat: CoordinateMatrix = new CoordinateMatrix(entries)

// 将坐标矩阵转换成2x2的分块矩阵并存储，尺寸通过参数传入
val matA: BlockMatrix = coordMat.toBlockMatrix(2,2).cache()

// 可以用validate()方法判断是否分块成功
matA.validate()

//构建成功后，可通过toLocalMatrix转换成本地矩阵，并查看其分块情况：
matA.toLocalMatrix

// 查看其分块情况
matA.numColBlocks
matA.numRowBlocks

// 计算矩阵A和其转置矩阵的积矩阵
val ata = matA.transpose.multiply(matA)
ata.toLocalMatrix
```

'''

分块矩阵BlockMatrix将矩阵分成一系列矩阵块，底层由矩阵块构成的RDD来进行数据存储。值得指出的是，用于生成分布式矩阵的底层RDD必须是已经确定（Deterministic）的，因为矩阵的尺寸将被存储下来，所以使用未确定的RDD将会导致错误。而且，不同类型的分布式矩阵之间的转换需要进行一个全局的shuffle操作，非常耗费资源。所以，根据数据本身的性质和应用需求来选取恰当的分布式矩阵存储类型是非常重要的。

## 坑

1． CoordinateMatrix 的columnSimilarities()方法用来计算每两列之间的余弦相似度，原始数据为n_31的矩阵，计算每两列的余弦相似度，理论上得到一个31_31的对称矩阵，对角线值为1（相同维度余弦相似度为1），及31\*31=961个值，实际得到的是一个上三角稀疏矩阵，只有465个值。 这是因为相似矩阵是一个对角线全为1的对称矩阵，为了节约空间，最后的结果省略了对角线，且只保存了一半。 故而实际为 （1+30）_31/2 = （31_31 -31）/2 = 465 2． BlockMatrix multiply求矩阵乘法时，官网上给出下面一段注释 ''' If other contains SparseMatrix, they willhave to be converted to a DenseMatrix.The output BlockMatrix will only consist of blocks of DenseMatrix. This maycause some performance issues until support for multiplying two sparse matricesis added. ''' 就是两个相乘的矩阵必须都是稠密的，因为结果中之会包含稠密矩阵的Block。但是其它几种矩阵的toBlockMatrix()方法，转成的都是稀疏矩阵。这里spark出现了自相矛盾的情况。 上述问题可以通过spark core实现，将小的矩阵做成广播变量，运行速度很快。解决了上述问题。

## 进阶--Marlin

Marlin 是南京大学 顾荣团队提出的基于Spark平台完成的矩阵运算库，为用户提供了大量矩阵运算的高层抽象原语，在性能方面远胜MapReduce相关的实现，在某些情况下甚至优于传统数据并行处理时代的MPI实现，Malin无论是在易用性还是性能方面都达到了一个很好的高度。 ![架构](http://pasa-bigdata.nju.edu.cn/projectImg/MarlinFram.bmp) 思想其实很简单，就是矩阵分块计算，而分块矩阵就成了小矩阵，然后就借助于Breeze实现。而对于Spark平台而言，其处理流程如下图： ![流程](https://img-blog.csdn.net/20151110203426434?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

## REF

* [spark分布式矩阵采坑记](https://blog.csdn.net/golden\_xuhaifeng/article/details/80600135)
* [Spark MLlib之矩阵](https://blog.csdn.net/qq\_33938256/article/details/52584964)
