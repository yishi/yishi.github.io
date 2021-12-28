---
title: kaggle 竞赛有感2
author: ''
date: '2017-07-31'
slug: kaggle-竞赛有感2
categories: []
tags: []
---
**小议python与R中个别模型默认参数的区别**

python中`ExtraTreesRegressor`的效果通常比`RandomForestRegressor`的效果好，伴随着变量的增加，两个模型的误差率不降反升.

经仔细对比，与默认参数设置有关，python中上面两个模型的默认使用所有特征变量构造分类回归树，R该模型的帮助文档中则介绍回归问题应该使用三分之一的特征构造基分类器，检验发现，改设为这个参数后，误差率有所下降。

下图中的python使用的全部的特征。
|     |python| R |
| ----| ----| ---- |
| random forest + log |0.43148	|0.43005|
|random forest + log top5 features	|0.44282|	0.41014|
|random forest +log + top5 + adverse_rush_hour + rush_hour |	0.44282|	0.40104|
|extremely randomized trees + log |	0.41573	|0.43255|
|extremely randomized trees +log + top5 features |	0.42089|	0.41237|

下图是python和R上面两个模型的默认参数表。

| |python |	R |	best|
| ---- | ---- | ---- | ---- |
|ramdom forest |	n_estimators : (default=10) The number of trees in the forest. | ntree=500	| 100/500 |
|ramdom forest |	max_features : (default=”auto”) If “auto”, then max_features=n_features |	mtry Number of variables randomly sampled as candidates at each split. for classification (sqrt(p) where p is number of variables in x) and regression (p/3)	| p/3 |
|extraTrees	|n_estimators=10	|ntree=500	|100 |
|extraTrees	| max_features default=”auto” If “auto”, then max_features=n_features | mtry: (default is ncol(x)/3 for regression and sqrt(ncol(x)) for classification)	|p/3|

备注：转移自新浪博客，截至2021年11月，原阅读数87，评论0个。  