---
title: Data Science Series in Python
author: ''
date: '2016-07-21'
slug: data-science-series-in-python
categories:
  - python
tags:
  - data science
---

[01 数据科学的基础——基本统计概念的介绍](https://nbviewer.org/github/yishi/Data-Science-Series-in-Python/blob/master/the_introduction_of_data_science_01.ipynb)

这部分介绍了一些基本的统计概念，主要是考虑到部分同学，一般对方差的实际意义很模糊，对中位数与平均数的适用场景不清晰，以及对箱线图和小提琴图的不熟悉。

+ 查看连续变量，使用直方图
+ 汇总连续变量，描述变量的集中位置，使用均值
+ 存在异常值的数据，新旧均值的对比，引入中位数
+ 汇总连续变量，描述变量的分散程度，引入方差、标准差的概念
+ 存在异常值的数据，引入四分位距，介绍箱线图
+ 大量增加异常值，汇总统计量不适用的场景，引入小提琴图。
 

[02 数据科学的模型——分类模型基础知识](https://nbviewer.org/github/yishi/Data-Science-Series-in-Python/blob/master/the_introduction_of_data_science_02.ipynb)

这部分以鸢尾花数据集为例

+ 先是简单的数据探索
+ 然后手动构建一个简单的分类器
+ 随后引入决策树模型
+ 又根据模型效果的思考，引入数据的拆分，如训练集和验证集的拆分，k重交叉检验的拆分
+ 随后介绍针对分类模型的评价方法，如混淆矩阵，precision，recall等指标，ROC曲线，PR曲线。
 

[03 数据科学的模型——分类模型简介](https://nbviewer.org/github/yishi/Data-Science-Series-in-Python/blob/master/the_introduction_of_data_science_03.ipynb)

这部分介绍了一些分类模型

+ 先介绍了逻辑回归的基本原理，用鸢尾花数据集画了分类边界；
+ 随后依次介绍了k最近邻算法、朴素贝叶斯分类器、支持向量机、集成算法（如装袋算法、随机森林、提升算法等）；
+ 最后以手写体数字数据集为例，使用各分类模型识别手写体数字，k最近邻算法效果最好。
 

[04 数据科学的模型——预测模型简介](https://nbviewer.org/github/yishi/Data-Science-Series-in-Python/blob/master/the_introduction_of_data_science_04.ipynb)

这部分介绍了一些回归模型。

+ 先介绍了一元线性回归，以婴儿出生月份与体重的数据为例进行讲解。
+ 介绍了回归模型处理高维数据的衍生模型，如岭回归、lasso、弹性网等。
+ 以糖尿病数据为例，进行预测分析。
+ 以波士顿房价数据为例，进行预测分析。
 

[05 数据科学的模型——聚类、降维、数据预处理及特征选择的简介](https://nbviewer.org/github/yishi/Data-Science-Series-in-Python/blob/master/the_introduction_of_data_science_05.ipynb)

这部分介绍了聚类、降维、数据预处理、特征选择的相关内容。

+ 以K-Means为例，介绍了聚类模型；
+ 以主成分分析PCA模型为例，介绍了降维模型；
+ 以数据标准化处理为例，介绍了数据预处理的相关方法；
+ 最后，介绍了特征选择的一些常用方法。

Welcome your advice and suggestion!

Just record, this article was posted at linkedin, and have 67 views to November 2021.

