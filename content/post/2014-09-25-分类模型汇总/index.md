---
title: 分类模型汇总
author: ''
date: '2014-09-25'
slug: 分类模型汇总
categories: []
tags: []
---

最近研究分類模型，找到幾篇文章，都是討論分類模型的性能的，都是使用的同一個公開數據，關於信用評級的，然後使用各個模型，使用各個指標評價這些模型的好壞，挺好的。

用R來跑模型的見《R語言與分類算法的績效評估》 http://blog.csdn.net/yujunbeta/article/details/18138957

用SAS跑模型的見下面的系列文章《分類模型的性能評估——以SAS Logistic回歸為例1-3》
+ http://cos.name/2008/12/measure-classification-model-performance-confusion-matrix/
+ http://cos.name/2008/12/measure-classification-model-performance-roc-auc/
+ http://cos.name/2009/02/measure-classification-model-performance-lift-gain/

R中的包caret是一個集成了各種算法的包，新包`caretEnsemble`是更強大，把caret包中的各種算法集成起來。比如算法隨機森林，是生成很多棵樹，把這些樹根據不同的權重組合起來，形成隨機森林，這個`caretEnsemble`是把隨機森林中的樹換成了各種算法，如邏輯回歸、支持向量機、神經網絡等等，把這些算法都組合起來組成一個超級算法吧。該包的詳細方法見下面的連接
http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.60.2859&rep=rep1&type=pdf
真的是天外有天，人外有人，站在巨人的肩膀上了。

PS: caret包的主頁http://topepo.github.io/caret/index.html 


备注：转移自新浪博客，截至2021年11月，原阅读数77，评论0个。

