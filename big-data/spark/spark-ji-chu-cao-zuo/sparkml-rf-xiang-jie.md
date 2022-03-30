# SparkML RF 详解

## RF参数

* checkpointInterval 检查间隔点，就是多少次迭代固化一次
* impurity 惩罚函数，可选值 entropy，gini,variance 默认gini
* maxbins 最大分箱数 默认32
* maxDepth 最大深度，默认5 最大值30
* numTrees 树数目，默认20
* setMaxDepth：最大树深度
* setMaxBins：最大装箱数，为了近似统计变量，比如变量有100个值，我只分成10段去做统计 讲一下MLlib中的bin和split概念，split为切割点，对应二叉的决策树，bin为桶或者箱子数，一个split把数据集划分成2个桶，所以bin是split的2倍。 setMinInstancesPerNode：每个节点最少实例
* setMinInfoGain：[最小信息增益](https://www.zhihu.com/question/22104055)
* setMaxMemoryInMB：最大内存MB单位，这个值越大，一次处理的节点划分就越多
* setCacheNodeIds：是否缓存节点id，缓存可以加速深层树的训练
* setCheckpointInterval：检查点间隔，就是多少次迭代固化一次
* setImpurity：随机森林有三种方式，entropy，gini,variance,回归肯定就是variance（残差）
  * 基尼不纯度,是指将来自集合中的某种结果随机应用在集合中，某一数据项的预期误差率。是在进行决策树编程的时候，对于混杂程度的预测中，一种度量方式。参考[熵和基尼不纯度的计算](http://onmyway-1985.iteye.com/blog/2083384)
  * 残差就是[方差](http://dataunion.org/5951.html)
* setSubsamplingRate：设置采样率，就是每次选多少比例的样本构成新树
* setSeed：采样种子，种子不变，采样结果不变
* setNumTrees：设置森林里有多少棵树
* setFeatureSubsetStrategy：设置特征子集选取策略，随机森林就是两个随机，构成树的样本随机，每棵树开始分裂的属性是随机的，其他跟决策树区别不大，注释这么写的

## 说明

模型的训练过程其实是决策树的构造过程，它采用自顶向下的递归方式，在决策树的内部结点进行属性值的比较，并根据不同的属性值判断从该结点向下分支，进行递归进行划分，直到满足一定的终止条件(可以进行自定义)，其中叶结点是要学习划分的类。在当前节点用哪个属性特征作为判断进行切分(也叫分裂规则)，取决于切分后节点数据集合中的类别(分区)的有序(纯)程度，划分后的分区数据越纯，那么当前分裂规则也合适。衡量节点数据集合的有序无序性，有熵、基尼Gini、方差，其中熵和Gini是针对分类的，方差是针对回归的。

RF最招人稀罕的地方就是基本不需要调参， 本质上它是bagging基础上做的CART， 所以如果减少了max\_features, 一般就不需要对子树的能力作限制了。然后，因为有随机因素，n\_estimators越多结果更稳定，所以只要允许内，数目越大越好。大致就是这样。

## REF

[决策树相关](https://www.cnblogs.com/fionacai/p/5894142.html) [spark RF 源码分析](http://blog.csdn.net/shenxiaoming77/article/details/51675242) [Sklearn-RandomForest随机森林](http://blog.csdn.net/cherdw/article/details/54971771) [sklearn官方API](http://scikit-learn.org/stable/modules/generated/sklearn.ensemble.RandomForestClassifier.html)
