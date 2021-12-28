---
title: 'Text Mining Series 3: Automatic Summarization'
author: ''
date: '2017-01-07'
slug: text-mining-series-3-automatic-summarization
categories:
  - python
  - R
tags:
  - text mining
---

The key technology we used in here, are page rank.

+ First of all, we split an article into sentences.

+ Second, we treat sentences as document, to do some data clean, tf-idf transformation, and document-term matrix, then multiply this dtm matrix and the transpose of dtm matrix, get the similar matrix between sentences.

+ Third, using above similar matrix as input, we take advantage of the function page.rank in igraph package to get the value of page rank as our text rank value. We sort the sentences by the value of page rank, then output the first five sentences as our summarization.

[Text Mining Series 3.1: Automatic Summarization using TextRank by Python](http://nbviewer.jupyter.org/github/yishi/Text-Mining-Series/blob/master/Text_mining_series_3.ipynb)

[Text Mining Series 3.2: Automatic Summarization using TextRank by R](https://github.com/yishi/my_R_code/blob/master/automatic%20summarization%20using%20text%20rank)

Welcome your advice and suggestion!

Just record, this article was posted at linkedin, and have 36 views to November 2021.
