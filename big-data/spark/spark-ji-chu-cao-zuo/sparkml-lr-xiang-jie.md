# SparkML LR详解

## 参数

* threshold: 如果label 1的概率>threshold，则预测为1，否则为0.
* regParam：正则项参数(相当于 λ)
* elasticNetParam：ElasticNet混合参数 α 。如果α=0, 惩罚项是一个L2 penalty。如果α=1，它是一个L1 penalty。如果0<α<1，则是L1和L2的结果。缺省为0，为L2罚项。
* maxIter: 缺省100次。
* tol：迭代收敛的容忍度。值越小，会得到更高精度，同时迭代次数开销也大。缺省为1E-6.
* fitIntercept: 是否要拟合截距项(intercept term)，缺省为true.
* standardization：在对模型进行fit前，是否对训练特征进行标准化(归一化)。模型的系数将总是返回到原比例(origin scale)上，它对用户是透明的。注意：当不使用正则化时，使用/不使用standardization，模型都应收敛到相同的解（solution）上。在R的GLMNET包里，缺省行为也为true。缺省为true。
* weightCol：如果没设置或空，所有实例为有为1的权重。如果设置，则相当于对unbalanced data设置不同的权重。缺省不设置。
* treeAggregate：如果特征维度或分区数太大，该参数需要调得更大。缺省为2.

## 正则

```
Lreg =  λ *[α * L1 + (1 - α ) * L2]

regPram = regParamL1+regParamL2
val regParamL1 = $(elasticNetParam) * $(regParam)
val regParamL2 = (1.0 - $(elasticNetParam)) * $(regParam)
```

两种正则化方法L1和L2。L2正则化假设模型参数服从高斯分布，L2正则化函数比L1更光滑，所以更容易计算；L1假设模型参数服从拉普拉斯分布，L1正则化具备产生稀疏解的功能，从而具备Feature Selection的能力。

## REF

* [MLlib 中ml lr 源码分析](http://d0evi1.com/spark-lr/)
* [LASSO-Logistic模型--基于R语言glmnet包](https://blog.csdn.net/yitianguxingjian/article/details/69874867)
* [Logistic回归参数设置，分类结果评估](https://www.2cto.com/net/201608/542278.html)
* [对比了下的spark mllib和 Liblinear 的LR的实现](https://blog.csdn.net/map\_lixiupeng/article/details/51814827)
