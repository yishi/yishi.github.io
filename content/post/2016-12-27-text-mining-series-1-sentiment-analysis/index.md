---
title: 'Text Mining Series 1: Sentiment Analysis'
author: ''
date: '2016-12-27'
slug: text-mining-series-1-sentiment-analysis
categories:
  - python
  - R
tags:
  - text mining
---

Several days ago, I discover a new package about text mining in R, **text2vec**, this package is great, I use this package and other packages to predict the defect name from the defect description text, this idea is similar to sentiment analysis, just change response variable Y values from positive/negative to defect name1/2/3/...

Below is a flowchart about text mining.

1. Text Data

2. Segmentation（for chinese text）jiebaR package

3. Transform to structural data, such as matrix, use text2vec package, English text could split the sentence to word in this package, which is similar to segmentation

3.1 Data Clean

+ first of all, delete stop words
+ second, choose the number of the word to make vector, n-gram
+ third, delete very common and very unusual terms
+ fourth, whether to use hashing trick, which could shorten the run time when there have large collections of documents
+ fifth, make a document-term matrix

3.2 Transform Document-term Matrix

+ normalization：decrease the influence of the lengths of the documents
+ tf-idf：increase the weight of terms which are specific and decrease the weight for terms used in most documents

4. Model

Such as sentiment analysis, use classification model, to predict positive or negative

Below is my R and Python code:

[Text Mining Series 1.1: Sentiment Analysis in Movie Review Dataset in English by R](https://github.com/yishi/my_R_code/blob/master/sentiment%20analysis%20for%20English%20text)

[Text Mining Series 1.2: Predict Defect Name From Defect Description Text in Chinese by R](https://github.com/yishi/my_R_code/blob/master/predict%20defect%20name%20for%20Chinese%20text)

[Text Mining Series 1.3: Predict Defect Name From Defect Description Text in Chinese by Python](https://nbviewer.org/github/yishi/Text-Mining-Series/blob/master/Text_mining_series_1.ipynb)

Welcome your advice and suggestion!

Just record, this article was posted at linkedin, and have 63 views to November 2021.
