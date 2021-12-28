---
title: 使用R语言制作具有交互效果的词云
author: ''
date: '2017-09-04'
slug: 使用r语言制作具有交互效果的词云
categories: []
tags: []
---

时过境迁，包也要更新了，中文分词用jiebaR，做词云用wordcloud2，该包是封装了javascript的某个词云包，因此，自带交互效果。

R code:
```{r}
library(wordcloud2)
library(jiebaR)

### get data 
data <- read.csv('C:\\Users\\your_file.csv', 
              stringsAsFactors = FALSE)
text <- data$包含文本的字段

###  load the engine 
### add stopword file 
cutter <- worker(byline = TRUE, 
 stop_word = "C:\\Users\\your_dir\\stop_words.txt")

# add new words
new_user_word(cutter, "不稳")

# split the sentences to words
word <- segment(text, cutter)
word_2 <- unlist(word)

# get word frequency
wordFreq <- sort(table(word_2), decreasing = TRUE)
df_freq <- data.frame(word = names(wordFreq),
                   freq = as.numeric(unname(wordFreq)))
# get wordcloud
wordcloud2(df_freq[1:100, ], fontFamily = 'Microsoft YaHei')
```

备注：转移自新浪博客，截至2021年11月，原阅读数99，评论0个。  

