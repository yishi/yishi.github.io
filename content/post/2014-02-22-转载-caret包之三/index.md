---
title: 转载——caret包之三
author: ''
date: '2014-02-22'
slug: 转载-caret包之三
categories:
  - R
tags: []
---

**caret包应用之三：建模与参数优化**

在进行建模时，需对模型的参数进行优化，在caret包中其主要函数命令是`train`。

+ 首先得到经过特征选择后的样本数据，并划分为训练样本和检验样本
```{r}
newdata4=newdata3[,Profile$optVariables]
inTrain = createDataPartition(mdrrClass, p = 3/4, list = FALSE)
trainx = newdata4[inTrain,]
testx = newdata4[-inTrain,]
trainy = mdrrClass[inTrain]
testy = mdrrClass[-inTrain]
```

+ 然后定义模型训练参数，`method`确定多次交叉检验的抽样方法，`number`确定了划分的重数，`repeats`确定了反复次数。
```{r}
fitControl = trainControl(method = "repeatedcv", number = 10, repeats = 3,returnResamp = "all")
```

+ 确定参数选择范围，本例建模准备使用`gbm`算法，相应的参数有如下三项:
```{r}
gbmGrid = expand.grid(.interaction.depth = c(1, 3),.n.trees = c(50, 100, 150, 200, 250, 300),.shrinkage = 0.1)
```

+ 利用`train`函数进行训练，使用的建模方法为提升决策树方法，
```{r}
gbmFit1 = train(trainx,trainy,method = "gbm",trControl = fitControl,tuneGrid = gbmGrid,verbose = FALSE)
```

+ 从结果可以观察到`interaction.depth`取1，`n.trees`取150时精度最高

+ 同样的图形观察
```{r}
plot(gbmFit1)
```

轉載自：http://xccds1977.blogspot.com/2011/09/caret_1976.html 

备注：转移自新浪博客，截至2021年11月，原阅读数153，评论0个。
