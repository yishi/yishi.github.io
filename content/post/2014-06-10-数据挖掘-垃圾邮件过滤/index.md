---
title: 数据挖掘——垃圾邮件过滤
author: ''
date: '2014-06-10'
slug: 数据挖掘-垃圾邮件过滤
categories: []
tags:
  - data science
---

来自《Machine learning for Hackers》第三章

原始数据来自  http://spamassassin.apache.org/publiccorpus/
作者的github网页也提供了数据和代码 https://github.com/johnmyleswhite/ML_for_Hackers 

数据是2002年、2003年和2005年的正常邮件，容易混为垃圾邮件的正常邮件和垃圾邮件三类。

思路很清晰：使用2002年的数据作为训练数据，2003年和2005年的数据作为测试数据。

在2002年的数据中，提取垃圾邮件的信件内容构建成词库，清理下詞庫（如刪除停用詞，刪除數字等），計算一些指標（如某個單詞的出現頻率、密度等），使用正常郵件的信件內容也構建類似的詞庫。

判別某新郵件是否為垃圾郵件：根据贝叶斯公式，計算新郵件中属于垃圾邮件或正常邮件的概率大小，如果属于垃圾邮件的概率大于属于正常邮件的概率，则判别为垃圾邮件。

关键就是如何使用贝叶斯公式？
这里使用到了朴素贝叶斯来进行分类，有关贝叶斯用于垃圾邮件判别的简介详见大牛博客： http://mindhacks.cn/2008/09/21/the-magical-bayesian-method/ 。

摘抄部分如下：

贝叶斯垃圾邮件过滤器

问题是什么？

问题是，给定一封邮件，判定它是否属于垃圾邮件。

按照先例，我们还是用 D 来表示这封邮件，注意 D 由 N 个单词组成。我们用 h+ 来表示垃圾邮件，h- 表示正常邮件。问题可以形式化地描述为求：

P(h+|D) = P(h+) * P(D|h+) / P(D)

P(h-|D) = P(h-) * P(D|h-) / P(D)

其中 P(h+) 和 P(h-) 这两个先验概率都是很容易求出来的，只需要计算一个邮件库里面垃圾邮件和正常邮件的比例就行了。

然而 P(D|h+) 却不容易求，因为 D 里面含有 N 个单词 d1, d2, d3, .. ，所以P(D|h+) = P(d1,d2,..,dn|h+) 。我们又一次遇到了数据稀疏性，为什么这么说呢？P(d1,d2,..,dn|h+) 就是说在垃圾邮件当中出现跟我们目前这封邮件一模一样的一封邮件的概率是多大！开玩笑，每封邮件都是不同的，世界上有无穷多封邮件。瞧，这就是数据稀疏 性，因为可以肯定地说，你收集的训练数据库不管里面含了多少封邮件，也不可能找出一封跟目前这封一模一样的。结果呢？我们又该如何来计算 P(d1,d2,..,dn|h+) 呢？

我们将 P(d1,d2,..,dn|h+)  扩展为： P(d1|h+) * P(d2|d1, h+) * P(d3|d2,d1, h+) * .. 。熟悉这个式子吗？这里我们会使用一个更激进的假设，我们假设 di 与 di-1 是完全条件无关的，于是式子就简化为 P(d1|h+) * P(d2|h+) * P(d3|h+) * .. 。这个就是所谓的条件独立假设，也正是朴素贝叶斯方法的朴素之处。而计算 P(d1|h+) * P(d2|h+) * P(d3|h+) * .. 就太简单了，只要统计 di 这个单词在垃圾邮件中出现的频率即可。

R代码如下：
```{r}
# 03-Classification/email_classify.R
# Load libraries
library('tm')
library('ggplot2')

# Set the global paths
spam.path <- file.path("spam")
spam2.path <- file.path("spam_2")
easyham.path <- file.path("easy_ham")
easyham2.path <- file.path("easy_ham_2")
hardham.path <- file.path("hard_ham")
hardham2.path <- file.path("hard_ham_2")

# Return a single element vector of just the email body
# This is a very simple approach, as we are only using 
# words as features
get.msg <- function(path)
{
  con <- file(path, open = "rt", encoding = "latin1")
  text <- readLines(con)
  # The message always begins after the first full line break
  if (length(which(text == "")) != 0 & which(text == "")[1] != length(text)) {
    msg <- text[seq(which(text == "")[1] + 1, length(text), 1)] 
  }else msg <- "" 
  close(con)
  return(paste(msg, collapse = "\n"))
}

# Get all the SPAM-y email into a single vector
spam.docs <- dir(spam.path)
spam.docs <- spam.docs[which(spam.docs != "cmds")]
all.spam <- sapply(spam.docs,
                  function(p) get.msg(file.path(spam.path, p)))

# Create a TermDocumentMatrix (TDM) from the corpus of SPAM email.
# The TDM control can be modified, and the sparsity level can be
# altered.  This TDM is used to create the feature set used to do
# train our classifier.
get.tdm <- function(doc.vec)
{
  control <- list(stopwords = TRUE,
                 removePunctuation = TRUE,
                 removeNumbers = TRUE,
                 minDocFreq = 2)
  doc.corpus <- Corpus(VectorSource(doc.vec))
  doc.dtm <- TermDocumentMatrix(doc.corpus, control)
  return(doc.dtm)
}

# Create a DocumentTermMatrix from that vector
spam.tdm <- get.tdm(all.spam)

# Create a data frame that provides the feature set from the training SPAM data
spam.matrix <- as.matrix(spam.tdm)
# 某個單詞在所有文檔中出現的總次數
spam.counts <- rowSums(spam.matrix)
spam.df <- data.frame(cbind(names(spam.counts),
                           as.numeric(spam.counts)),
                     stringsAsFactors = FALSE)
names(spam.df) <- c("term", "frequency")
spam.df$frequency <- as.numeric(spam.df$frequency)
# 某個單詞在文檔中出現的頻率，即文檔的覆蓋率吧
spam.occurrence <- sapply(1:nrow(spam.matrix),
                         function(i)
                         {
                           length(which(spam.matrix[i, ] > 0)) / ncol(spam.matrix)
                         })
# 某個單詞出現的次數占所有單詞的總次數的比例
spam.density <- spam.df$frequency / sum(spam.df$frequency)

# Add the term density and occurrence rate
spam.df <- transform(spam.df,
                    density = spam.density,
                    occurrence = spam.occurrence)

head(spam.df[order(spam.df$occurrence, decreasing = T), ])
head(spam.df[with(spam.df, order(-occurrence)), ])
#       term frequency    density occurrence
# 6139  email      728 0.006301066     0.552
# 15024 please      378 0.003271708     0.466
# 11786  list      385 0.003332295     0.414
# 2285   body      355 0.003072635     0.390
# 22012  will      714 0.006179892     0.386
# 9245   html      390 0.003375571     0.374

# Now do the same for the EASY HAM email
easyham.docs <- dir(easyham.path)
easyham.docs <- easyham.docs[which(easyham.docs != "cmds")]
all.easyham <- sapply(easyham.docs[1:length(spam.docs)],
                     function(p) get.msg(file.path(easyham.path, p)))

easyham.tdm <- get.tdm(all.easyham)

easyham.matrix <- as.matrix(easyham.tdm)
easyham.counts <- rowSums(easyham.matrix)
easyham.df <- data.frame(cbind(names(easyham.counts),
                              as.numeric(easyham.counts)),
                        stringsAsFactors = FALSE)
names(easyham.df) <- c("term", "frequency")
easyham.df$frequency <- as.numeric(easyham.df$frequency)
easyham.occurrence <- sapply(1:nrow(easyham.matrix),
                            function(i)
                            {
                              length(which(easyham.matrix[i, ] > 0)) / ncol(easyham.matrix)
                            })
easyham.density <- easyham.df$frequency / sum(easyham.df$frequency)

easyham.df <- transform(easyham.df,
                       density = easyham.density,
                       occurrence = easyham.occurrence)

head(easyham.df[order(easyham.df$occurrence, decreasing = T), ])
#       term frequency    density occurrence
# 6615  list      289 0.004782630     0.430
# 12400 wrote      227 0.003756599     0.370
# 4716 group      217 0.003591110     0.358
# 11811  use      252 0.004170321     0.358
# 1476   can      297 0.004915021     0.344
# 6196  just      250 0.004137223     0.324

# This is the our workhorse function for classifying email.  It takes
# two required paramters: a file path to an email to classify, and
# a data frame of the trained data.  The function also takes two
# optional parameters.  First, a prior over the probability that an email
# is SPAM, which we set to 0.5 (naive), and constant value for the
# probability on words in the email that are not in our training data.
# The function returns the naive Bayes probability that the given email
# is SPAM. 
classify.email <- function(path, training.df, prior = 0.5, c = 1e-6)
{
  # Here, we use many of the support functions to get the
  # email text data in a workable format
  msg <- get.msg(path)
  msg.tdm <- get.tdm(msg)
  msg.freq <- rowSums(as.matrix(msg.tdm))
  # Find intersections of words
  msg.match <- intersect(names(msg.freq), training.df$term)
  # Now, we just perform the naive Bayes calculation
  if(length(msg.match) < 1)
  {
    return(prior * c ^ (length(msg.freq)))
  }
  else
  {
    match.probs <- training.df$occurrence[match(msg.match, training.df$term)]
    return(prior * prod(match.probs) * c ^ (length(msg.freq) - length(msg.match)))
  }
}

# Run classifer against HARD HAM
hardham.docs <- dir(hardham.path)
hardham.docs <- hardham.docs[which(hardham.docs != "cmds")]

# path <- file.path(hardham.path, hardham.docs[1])

hardham.spamtest <- sapply(hardham.docs,
                          function(p) classify.email(file.path(hardham.path, p), training.df = spam.df))

hardham.hamtest <- sapply(hardham.docs,
                         function(p) classify.email(file.path(hardham.path, p), training.df = easyham.df))

hardham.res <- ifelse(hardham.spamtest > hardham.hamtest,
                     TRUE,
                     FALSE)
summary(hardham.res)

# Finally, attempt to classify the HARDHAM data using the classifer developed above.
# The rule is to classify a message as SPAM if Pr(email) = SPAM > Pr(email) = HAM

spam.classifier <- function(path)
{
  pr.spam <- classify.email(path, spam.df)
  pr.ham <- classify.email(path, easyham.df)
  return(c(pr.spam, pr.ham, ifelse(pr.spam > pr.ham, 1, 0)))
}

spam.classifier <- function(path)
{
  pr.spam <- classify.email(path, spam.df, prior = 0.2)
  pr.ham <- classify.email(path, easyham.df, prior = 0.8)
  return(c(pr.spam, pr.ham, ifelse(pr.spam > pr.ham, 1, 0)))
}

class.df <- suppressWarnings(lapply(hardham.docs,
                                         function(p)
                                         {
                                           spam.classifier(file.path(hardham.path, p))
                                         }))
head(class.df)

################# test classifier ##############
# Get lists of all the email messages
easyham2.docs <- dir(easyham2.path)
easyham2.docs <- easyham2.docs[which(easyham2.docs != "cmds")]

hardham2.docs <- dir(hardham2.path)
hardham2.docs <- hardham2.docs[which(hardham2.docs != "cmds")]

spam2.docs <- dir(spam2.path)
spam2.docs <- spam2.docs[which(spam2.docs != "cmds")]

# Classify them all!
easyham2.class <- suppressWarnings(lapply(easyham2.docs,
                                         function(p)
                                         {
                                           spam.classifier(file.path(easyham2.path, p))
                                         }))
hardham2.class <- suppressWarnings(lapply(hardham2.docs,
                                         function(p)
                                         {
                                           spam.classifier(file.path(hardham2.path, p))
                                         }))
spam2.class <- suppressWarnings(lapply(spam2.docs,
                                      function(p)
                                      {
                                        spam.classifier(file.path(spam2.path, p))
                                      }))
# Create a single, final, data frame with all of the classification data in it
easyham2.matrix <- do.call(rbind, easyham2.class)
easyham2.final <- cbind(easyham2.matrix, "EASYHAM")

hardham2.matrix <- do.call(rbind, hardham2.class)
hardham2.final <- cbind(hardham2.matrix, "HARDHAM")

spam2.matrix <- do.call(rbind, spam2.class)
spam2.final <- cbind(spam2.matrix, "SPAM")

class.matrix <- rbind(easyham2.final, hardham2.final, spam2.final)
class.df <- data.frame(class.matrix, stringsAsFactors = FALSE)
names(class.df) <- c("Pr.SPAM" ,"Pr.HAM", "Class", "Type")
class.df$Pr.SPAM <- as.numeric(class.df$Pr.SPAM)
class.df$Pr.HAM <- as.numeric(class.df$Pr.HAM)
class.df$Class <- as.logical(as.numeric(class.df$Class))
class.df$Type <- as.factor(class.df$Type)
head(class.df)

get.results <- function(bool.vector)
{
  results <- c(length(bool.vector[which(bool.vector == FALSE)]) / length(bool.vector),
              length(bool.vector[which(bool.vector == TRUE)]) / length(bool.vector))
  return(results)
}

# Save results as a 2x3 table
easyham2.col <- get.results(subset(class.df, Type == "EASYHAM")$Class)
hardham2.col <- get.results(subset(class.df, Type == "HARDHAM")$Class)
spam2.col <- get.results(subset(class.df, Type == "SPAM")$Class)

class.res <- rbind(easyham2.col, hardham2.col, spam2.col)
colnames(class.res) <- c("NOT SPAM", "SPAM")
print(class.res)
```
备注：转移自新浪博客，截至2021年11月，原阅读数409，评论0个。


