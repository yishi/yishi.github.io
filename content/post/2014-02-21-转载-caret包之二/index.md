---
title: 转载——caret包之二
author: ''
date: '2014-02-21'
slug: 转载-caret包之二
categories: []
tags: []
---

轉載網址：http://xccds1977.blogspot.com/2011/09/caret_24.html 

**caret包应用之二：特征选择**

在进行数据挖掘时，我们并不需要将所有的自变量用来建模，而是从中选择若干最重要的变量，这称为特征选择（feature selection）。

一种算法就是后向选择，即先将所有的变量都包括在模型中，然后计算其效能（如误差、预测精度）和变量重要排序，然后保留最重要的若干变量，再次计算效能，这样反复迭代，找出合适的自变量数目。

这种算法的一个缺点在于可能会存在过度拟合，所以需要在此算法外再套上一个样本划分的循环。 在caret包中的`rfe`命令可以完成这项任务。


+ 首先定义几个整数，程序必须测试这些数目的自变量.
```{r}
subsets = c(20,30,40,50,60,70,80)
```

+ 然后定义控制参数，`functions`是确定用什么样的模型进行自变量排序.

本例选择的模型是随机森林即`rfFuncs`，可以选择的还有`lmFuncs`（线性回归），`nbFuncs`（朴素贝叶斯），`treebagFuncs`（装袋决策树），`caretFuncs`（自定义的训练模型）。

+ `method`是确定用什么样的抽样方法.

本例使用`cv`即交叉检验, 还有提升`boot`以及留一交叉检验`LOOCV`
```{r}
ctrl= rfeControl(functions = rfFuncs, method = "cv",verbose = FALSE, returnResamp = "final")
```

+ 最后使用`rfe`命令进行特征选择，计算量很大，这得花点时间。
```{r}
Profile = rfe(newdata3, mdrrClass, sizes = subsets, rfeControl = ctrl)
```

+ 观察结果选择50个自变量时，其预测精度最高。
```{r}
print(Profile)
```

+ 用图形也可以观察到同样结果
```{r}
plot(Profile)
```

+ 下面的命令则可以返回最终保留的自变量
```{r}
Profile$optVariables
```

备注：转移自新浪博客，截至2021年11月，原阅读数122，评论0个。

