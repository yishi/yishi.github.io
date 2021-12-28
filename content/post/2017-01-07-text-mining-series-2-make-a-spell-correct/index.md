---
title: 'Text Mining Series 2: Make a Spell Correct'
author: ''
date: '2017-01-07'
slug: text-mining-series-2-make-a-spell-correct
categories:
  - python
  - R
tags:
  - text mining
---

When you input a wrong word, Google will give us some right words, this is very interesting.

The main idea is:

+ First of all, for your input word, spelling corrector will change that word by delete one letter in any place, or transpose one letter in any place with any letter, or replace one letter in any place by any letter, or insert any letter.

+ Second, the new word vector, which created by delete, transpose, replace, insert the input word, will be checked in the dictionary, which include correct words.

+ Third, look for the most common and the most similar right words,then output them.

More details you could refer the link [here](http://norvig.com/spell-correct.html).

[Text Mining Series 2.1: Make a Spell Checker for English Text by Python](https://nbviewer.org/github/yishi/Text-Mining-Series/blob/master/Text_mining_series_2.ipynb)

[Text Mining Series 2.2: Make a Spell Checker for English and Chinese Text by R](https://github.com/yishi/my_R_code/blob/master/spell%20checker%20for%20English%20and%20Chinese%20Text)

Welcome your advice and suggestion!

Just record, this article was posted at linkedin, and have 49 views to November 2021.