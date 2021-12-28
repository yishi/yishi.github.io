---
title: 转载——caret包初探
author: ''
date: '2014-02-20'
slug: 轉載-caret包初探
categories:
  - R
tags: []
---

最近看caret包開發者的書《Applied Predictive Modeling》，剛好下面牛人的文章是一個很好的匯總，就不再重複匯總了。

轉載自http://xccds1977.blogspot.com/2011/09/caret.html

**caret包应用之一：数据预处理**

在进行数据挖掘时，我们会用到R中的很多扩展包，各自有不同的函数和功能。如果能将它们综合起来应用就会很方便。

caret包（Classification and Regression Training）就是为了解决分类和回归问题的数据训练而创建的一个综合工具包。下面的例子围绕数据挖掘的几个核心步骤来说明其应用。

本例涉及到的数据是一个医学实验数据，载入数据之后可以发现其样本数为528，自变量数为342，mdrrDescr为自变量数据框，mdrrClass为因变量。
```{r}
library(caret)

data(mdrr)
```
本例的样本数据所涉及到的变量非常多，需要对变量进行初步降维。
+ 其中一种需要删除的变量是常数自变量，或者是方差极小的自变量，对应的命令是`nearZeroVar`，可以看到新数据集的自变量减少到了297个。
```{r}
zerovar=nearZeroVar(mdrrDescr)
newdata1=mdrrDescr[,-zerovar]
```
+ 另一类需要删除的是与其它自变量有很强相关性的变量，对应的命令是`findcorrelation`。

+ 自变量中还有可能存在多重共线性问题，可以用`findLinearCombos`命令将它们找出来。这样处理后自变量减少为94个。
```{r}
descrCorr = cor(newdata1)
highCorr = findCorrelation(descrCorr, 0.90)
newdata2 = newdata1[, -highCorr]
comboInfo = findLinearCombos(newdata2)
newdata2=newdata2[, -comboInfo$remove]
```
+ 我们还需要将数据进行标准化并补足缺失值.

这时可以用`preProcess`命令，缺省参数是标准化数据，其高级功能还包括用K近邻和装袋决策树两种方法来预测缺失值。此外它还可以进行cox幂变换和主成分提取。
```{r}
Process = preProcess(newdata2)
newdata3 = predict(Process, newdata2)
```

+ 最后是用`createDataPartition`将数据进行划分，分成75%的训练样本和25%检验样本，类似的命令还包括了`createResample`用来进行简单的自助法抽样，还有`createFolds`来生成多重交叉检验样本。
```{r}
inTrain = createDataPartition(mdrrClass, p = 3/4, list = FALSE)
trainx = newdata3[inTrain,]
testx = newdata3[-inTrain,]
trainy = mdrrClass[inTrain]
testy = mdrrClass[-inTrain]
```

+ 在建模前还可以对样本数据进行图形观察，例如对前两个变量绘制箱线图
```{r}
featurePlot(trainx[,1:2],trainy,plot='box')
```

备注：转移自新浪博客，截至2021年11月，原阅读数149，评论0个。


