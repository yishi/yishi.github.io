---
title: 'Data In Action Series 1: Recognition of Handwritten Digits'
author: ''
date: '2016-07-20'
categories:
  - python
tags:
  - computer_vision
slug: data-in-action-series-1-recognition-of-handwritten-digits
---

The handwritten digits data set which we used is made up of 1797 8x8 images. 

Each image is of a hand-written digit. 

![](images/1.jpg)

Each pixel could be showed as a number.

![](images/2.jpg)

In order to utilize an 8x8 figure like this, we'd have to first transform it into a feature vector with length 64.

![](images/3.jpg)

Now, we transform a figure form to an array which has 64 features vector.

We could use all the classification algorithm to recognizing the handwritten digits.

![](images/4.jpg)

Because of the shape of handwritten digits are complex, we find out the k nearest neighbors model have the best effect, and support vector machine, which kernel is polynomial, also have a good effect.

Please refer the whole python code in [here](https://nbviewer.org/github/yishi/Data-In-Action-Series-in-Python/blob/master/data_in_action_series_1.ipynb).

Welcome your advice and suggestion!

Just record, this article was posted at linkedin, and have 31 views to November 2021.