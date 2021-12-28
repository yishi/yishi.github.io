---
title: 数据挖掘——邮件优先级别
author: ''
date: '2014-06-11'
slug: 数据挖掘-邮件优先级别
---

来自《Machine learning for Hackers》第四章

原始数据来自  http://spamassassin.apache.org/publiccorpus/

作者的github网页也提供了数据和代码 https://github.com/johnmyleswhite/ML_for_Hackers 

继上文的**判别邮件是否垃圾邮件后，对正常邮件的优先级别进行排序，对优先级别高的邮件突出显示**。

+ 首先：提取邮件的全文、发送者、主题、发送日期、邮件编号（方便以后通过编号查找原始邮件）

+ 其次：计算几个权重。如：发送者的权重（对发送者的出现次数取对数）、活跃主题的权重（活跃主题中发信者的权重、对活跃主题的频率与时间间隔的比值取对数得到的权重、活跃主题中关键字的权重）、邮件正文中关键字的权重。

+ 最后：用训练集数据计算上述的5个权重，5个权重的乘积为新邮件的总权重，用训练集中的总权重的中位数作为是否优先的判断标准，用测试集计算每封邮件的总权重，根据是否大于标准，确定是否推荐。

R代码见下：
```{r}
# 第一階段： 提取郵件的發送者、主題、全文、發送日期、郵件編號
# Set the global paths
easyham.path <- file.path("easy_ham")
# easyham2.path <- file.path("easy_ham_2")

easyham.docs <- dir(easyham.path)
easyham.docs <- easyham.docs[which(easyham.docs != "cmds")]

# path <- file.path(easyham.path, easyham.docs[1])

# We define a set of function that will extract the data
# for the feature set we have defined to rank email
# impportance.  This includes the following: message
# body, message source, message subject, and date the
# message was sent.

# Simply returns the full text of a given email message
msg.full <- function(path)
{
  con <- file(path, open = "rt", encoding = "latin1")
  msg <- readLines(con)
  close(con)
  return(msg)
}

# Retuns the email address of the sender for a given
# email message
get.from <- function(msg.vec)
{
  from <- msg.vec[grepl("From: ", msg.vec)]
  from <- strsplit(from, '[":<> ]')[[1]]
  from <- from[which(from  != "" & from != " ")]
  return(from[grepl("@", from)][1])
}

# Retuns the subject string for a given email message
get.subject <- function(msg.vec)
{
  subj <- msg.vec[grepl("Subject: ", msg.vec)]
  if(length(subj) > 0)
  {
   return(strsplit(subj, "Subject: ")[[1]][2])
  }
  else
  {
   return("")
  }
}

# Similar to the function from Chapter 3, this returns
# only the message body for a given email.
get.msg <- function(msg.vec)
{
  if (length(which(msg.vec == "")) != 0 & which(msg.vec == "")[1] != length(msg.vec)) {
    msg <- msg.vec[seq(which(msg.vec == "")[1] + 1, length(msg.vec), 1)]
  }else msg <- ""
  return(paste(msg, collapse = "\n"))
}

# Retuns the date a given email message was received
get.date <- function(msg.vec)
{
  date.grep <- grepl("^Date: ", msg.vec)
  date.grep <- which(date.grep == TRUE)
  date <- msg.vec[date.grep[1]]
  date <- strsplit(date, "\\+|\\-|: ")[[1]][2]
  # substitute any leading or trailing whitespace
  # in the character string
  date <- gsub("^\\s+|\\s+$", "", date)
  # trim off any characters after a 25-character limit
  return(strtrim(date, 25))
}

# This function ties all of the above helper functions together.
# It returns a vector of data containing the feature set
# used to categorize data as priority or normal HAM
parse.email <- function(path)
{
  full.msg <- msg.full(path)
  date <- get.date(full.msg)
  from <- get.from(full.msg)
  subj <- get.subject(full.msg)
  msg <- get.msg(full.msg)
  return(c(date, from, subj, msg, path))
}

# In this case we are not interested in classifiying SPAM or HAM, so we will take
# it as given that is is being performed.  As such, we will use the EASY HAM email
# to train and test our ranker.

easyham.parse <- lapply(easyham.docs,
                       function(p) parse.email(file.path(easyham.path, p)))

# Convert raw data from list to data frame
ehparse.matrix <- do.call(rbind, easyham.parse)
allparse.df <- data.frame(ehparse.matrix, stringsAsFactors = FALSE)
names(allparse.df) <- c("Date", "From.EMail", "Subject", "Message", "Path")

# 數據整理，把數據分兩部份，前面是訓練集，後面是測試集
 Convert date strings to POSIX for comparison. Because the emails data
# contain slightly different date format pattners we have to account for
# this by passining them as required partmeters of the function.
date.converter <- function(dates, pattern1, pattern2)
{
  pattern1.convert <- strptime(dates, pattern1)
  pattern2.convert <- strptime(dates, pattern2)
  pattern1.convert[is.na(pattern1.convert)] <- pattern2.convert[is.na(pattern1.convert)]
  return(pattern1.convert)
}

pattern1 <- "%a, %d %b %Y %H:%M:%S"
pattern2 <- "%d %b %Y %H:%M:%S"

Sys.getlocale("LC_TIME")
Sys.setlocale("LC_TIME", "C")

allparse.df$Date <- date.converter(allparse.df$Date, pattern1, pattern2)

# Convert emails and subjects to lower-case
allparse.df$Subject <- tolower(allparse.df$Subject)
allparse.df$From.EMail <- tolower(allparse.df$From.EMail)

# Order the messages chronologically
priority.df <- allparse.df[with(allparse.df, order(Date)), ]

# We will use the first half of the priority.df to train our priority in-box algorithm.
# Later, we will use the second half to test.
priority.train <- priority.df[1:(round(nrow(priority.df) / 2)), ]

# 第二階段： 計算各種權重
# 計算發信者的權重
# The first step is to create rank weightings for all of the features.
# We begin with the simpliest: who the email is from.

# Calculate the frequency of correspondence with all emailers in the training set
tmp <- priority.train$Date
priority.train$Date <- as.character(priority.train$Date)
from.weight2 <- ddply(priority.train, .(From.EMail), summarise, Freq = length(Subject))
priority.train$Date <- tmp
rm(tmp)

library(reshape2)
from.weight <- melt(with(priority.train, table(From.EMail)),
                   value.name="Freq")

from.weight <- from.weight[with(from.weight, order(Freq)), ]

# We take a subset of the from.weight data frame to show our most frequent
# correspondents.
from.ex <- subset(from.weight, Freq > 6)

from.scales <- ggplot(from.ex) +
  geom_rect(aes(xmin = 1:nrow(from.ex) - 0.5,
               xmax = 1:nrow(from.ex) + 0.5,
               ymin = 0,
               ymax = Freq,
               fill = "lightgrey",
               color = "darkblue")) +
  scale_x_continuous(breaks = 1:nrow(from.ex), labels = from.ex$From.EMail) +
  coord_flip() +
  scale_fill_manual(values = c("lightgrey" = "lightgrey"), guide = "none") +
  scale_color_manual(values = c("darkblue" = "darkblue"), guide = "none") +
  ylab("Number of Emails Received (truncated at 6)") +
  xlab("Sender Address") +
  theme_bw() +
  theme(axis.text.y = element_text(size = 5, hjust = 1))


# Log weight scheme, very simple but effective
from.weight <- transform(from.weight,
                        Weight = log(Freq + 1),
                        log10Weight = log10(Freq + 1))

from.rescaled <- ggplot(from.weight, aes(x = 1:nrow(from.weight))) +
  geom_line(aes(y = Weight, linetype = "ln")) +
  geom_line(aes(y = log10Weight, linetype = "log10")) +
  geom_line(aes(y = Freq, linetype = "Absolute")) +
  scale_linetype_manual(values = c("ln" = 1,
                                  "log10" = 2,
                                  "Absolute" = 3),
                       name = "Scaling") +
  xlab("") +
  ylab("Number of emails Receieved") +
  theme_bw() +
  theme(axis.text.y = element_blank(), axis.text.x = element_blank())

# 活躍主題的權重
# To calculate the rank priority of an email we should calculate some probability that
# the user will respond to it.  In our case, we only have one-way communication data.
# In this case, we can calculate a weighting based on words in threads that have a lot
# of activity.

# This function is used to find threads within the data set.  The obvious approach
# here is to use the 're:' cue from the subject line to identify message threads.
# 找到活躍的主題
find.threads <- function(email.df)
{
  response.threads <- strsplit(email.df$Subject, "re: ")
  is.thread <- sapply(response.threads,
                     function(subj) ifelse(subj[1] == "", TRUE, FALSE))
  threads <- response.threads[is.thread]
  senders <- email.df$From.EMail[is.thread]
  threads <- sapply(threads,
                   function(t) paste(t[2:length(t)], collapse = "re: "))
  return(cbind(senders,threads))
}
# email.df <- priority.train
threads.matrix <- find.threads(priority.train)

# Using the matrix of threads generated by the find.threads function this function
# creates a data from of the sender's email, the frequency of emails from that
# sender, and a log-weight for that sender based on the freqeuncy of corresponence.
# 活躍主題中發送者及其權重
email.thread <- function(threads.matrix)
{
  senders <- threads.matrix[, 1]
  senders.freq <- table(senders)
  senders.matrix <- cbind(names(senders.freq),
                         senders.freq,
                         log(senders.freq + 1))
  senders.df <- data.frame(senders.matrix, stringsAsFactors=FALSE)
  row.names(senders.df) <- 1:nrow(senders.df)
  names(senders.df) <- c("From.EMail", "Freq", "Weight")
  senders.df$Freq <- as.numeric(senders.df$Freq)
  senders.df$Weight <- as.numeric(senders.df$Weight)
  return(senders.df)
}

senders.df <- email.thread(threads.matrix)

# As an additional weight, we can enhance our notion of a thread's importance
# by measuring the time between responses for a given email.  This function
# takes a given thread and the email.df data frame to generate a weighting
# based on this activity level.  This function returns a vector of thread
# activity, the time span of a thread, and its log-weight.
# 活躍主題的頻率、時間間隔，兩種的比值取對數后得到的權重
thread.counts <- function(thread, email.df)
{
  # Need to check that we are not looking at the original message in a thread,
  # so we check the subjects against the 're:' cue.
  thread.times <- email.df$Date[which(email.df$Subject == thread |
                                       email.df$Subject == paste("re:", thread))]
  freq <- length(thread.times)
  min.time <- min(thread.times)
  max.time <- max(thread.times)
  time.span <- as.numeric(difftime(max.time, min.time, units = "secs"))
  if(freq < 2)
  {
    return(c(NA, NA, NA))
  }
  else
  {
    trans.weight <- freq / time.span
   log.trans.weight <- 10 + log(trans.weight, base = 10)
   return(c(freq, time.span, log.trans.weight))
  }
}
# email.df <- priority.train
# thread <- threads.matrix[, 2][1]

# This function uses the threads.counts function to generate a weights
# for all email threads.
get.threads <- function(threads.matrix, email.df)
{
  threads <- unique(threads.matrix[, 2])
  thread.counts <- lapply(threads,
                         function(t) thread.counts(t, email.df))
  thread.matrix <- do.call(rbind, thread.counts)
  return(cbind(threads, thread.matrix))
}

# Now, we put all of these function to work to generate a training set
# based on our thread features.
thread.weights <- get.threads(threads.matrix, priority.train)
thread.weights <- data.frame(thread.weights, stringsAsFactors = FALSE)
names(thread.weights) <- c("Thread", "Freq", "Response", "Weight")
thread.weights$Freq <- as.numeric(thread.weights$Freq)
thread.weights$Response <- as.numeric(thread.weights$Response)
thread.weights$Weight <- as.numeric(thread.weights$Weight)
thread.weights <- subset(thread.weights, is.na(thread.weights$Freq) == FALSE)

# 活躍主題中關鍵字的權重
# Similar to what we did in Chapter 3, we create a simple function to return a
# vector of word counts.  This time, however, we keep the TDM as a free
# parameter of the function.
term.counts <- function(term.vec, control)
{
  vec.corpus <- Corpus(VectorSource(term.vec))
  vec.tdm <- TermDocumentMatrix(vec.corpus, control = control)
  return(rowSums(as.matrix(vec.tdm)))
}

thread.terms <- term.counts(thread.weights$Thread,
                           control = list(stopwords = TRUE))
thread.terms <- names(thread.terms)

term.weights <- sapply(thread.terms,
                      function(t) mean(thread.weights$Weight[grepl(t, thread.weights$Thread, fixed = TRUE)]))
term.weights <- data.frame(list(Term = names(term.weights),
                               Weight = term.weights),
                          stringsAsFactors = FALSE,
                          row.names = 1:length(term.weights))

# 郵件正文中關鍵字的權重
# Finally, create weighting based on frequency of terms in email.
# Will be similar to SPAM detection, but in this case weighting
# high words that are particularly HAMMMY.

msg.terms <- term.counts(priority.train$Message,
                        control = list(stopwords = TRUE,
                                       removePunctuation = TRUE,
                                       removeNumbers = TRUE))
msg.weights <- data.frame(list(Term = names(msg.terms),
                              Weight = log(msg.terms, base = 10)),
                         stringsAsFactors = FALSE,
                         row.names = 1:length(msg.terms))

# Remove words that have a zero weight
msg.weights <- subset(msg.weights, Weight > 0)

# 第三階段：計算訓練集中郵件的優先級
# This function uses our pre-calculated weight data frames to look up
# the appropriate weightt for a given search.term.  We use the 'term'
# parameter to dertermine if we are looking up a word in the weight.df
# for it message body weighting, or for its subject line weighting.
get.weights <- function(search.term, weight.df, term = TRUE)
{
  if(length(search.term) > 0)
  {
   if(term)
    {
     term.match <- match(names(search.term), weight.df$Term)
    }
    else
    {
     term.match <- match(search.term, weight.df$Thread)
    }
   match.weights <- weight.df$Weight[which(!is.na(term.match))]
   if(length(match.weights) < 1)
    {
     return(1)
    }
    else
    {
     return(mean(match.weights))
    }
  }
  else
  {
   return(1)
  }
}

# search.term <- msg.terms
# weight.df <- msg.weights

# Our final step is to write a function that will assign a weight to each message based
# on all of our, we create a function that will assign a weight to each message based on
# the mean weighting across our entire feature set.
rank.message <- function(path)
{
  msg <- parse.email(path)
  # Weighting based on message author
 
  # First is just on the total frequency
  from <- ifelse(length(which(from.weight$From.EMail == msg[2])) > 0,
                from.weight$Weight[which(from.weight$From.EMail == msg[2])],
                1)
 
  # Second is based on senders in threads, and threads themselves
  thread.from <- ifelse(length(which(senders.df$From.EMail == msg[2])) > 0,
                       senders.df$Weight[which(senders.df$From.EMail == msg[2])],
                       1)
 
  subj <- strsplit(tolower(msg[3]), "re: ")
  is.thread <- ifelse(subj[[1]][1] == "", TRUE, FALSE)
  if(is.thread)
  {
    activity <- get.weights(subj[[1]][2], thread.weights, term = FALSE)
  }else{
    activity <- 1
  }
 
  # Next, weight based on terms   
 
  # Weight based on terms in threads
  thread.terms <- term.counts(msg[3], control = list(stopwords = TRUE))
  thread.terms.weights <- get.weights(thread.terms, term.weights)
 
  # Weight based terms in all messages
  msg.terms.one <- term.counts(msg[4],
                          control = list(stopwords = TRUE,
                                         removePunctuation = TRUE,
                                         removeNumbers = TRUE))
  msg.weight.one <- get.weights(msg.terms.one, msg.weights)
 
  # Calculate rank by interacting all weights
  rank <- prod(from,
              thread.from,
              activity,
              thread.terms.weights,
              msg.weight.one)
 
  return(c(msg[1], msg[2], msg[3], rank))
}

# Find splits again
train.paths <- priority.df$Path[1:(round(nrow(priority.df) / 2))]
test.paths <- priority.df$Path[((round(nrow(priority.df) / 2)) + 1):nrow(priority.df)]


# priority.train <- priority.df[1:(round(nrow(priority.df) / 2)), ]

# Now, create a full-featured training set.
train.ranks <- suppressWarnings(lapply(train.paths, rank.message))
train.ranks.matrix <- do.call(rbind, train.ranks)
train.ranks.matrix <- cbind(train.paths, train.ranks.matrix, "TRAINING")
train.ranks.df <- data.frame(train.ranks.matrix, stringsAsFactors = FALSE)
names(train.ranks.df) <- c("Message", "Date", "From", "Subj", "Rank", "Type")
train.ranks.df$Rank <- as.numeric(train.ranks.df$Rank)

# Set the priority threshold to the median of all ranks weights
priority.threshold <- median(train.ranks.df$Rank)

# Visualize the results to locate threshold
threshold.plot <- ggplot(train.ranks.df, aes(x = Rank)) +
  stat_density(aes(fill="darkred")) +
  geom_vline(xintercept = priority.threshold, linetype = 2) +
  scale_fill_manual(values = c("darkred" = "darkred"), guide = "none") +
  theme_bw()

# Classify as priority, or not
train.ranks.df$Priority <- ifelse(train.ranks.df$Rank >= priority.threshold, 1, 0)

# 第四階段 ： 計算測試集中郵件的優先度，確定是否推薦
# Now, test our ranker by performing the exact same procedure on the test data
test.ranks <- suppressWarnings(lapply(test.paths,rank.message))
test.ranks.matrix <- do.call(rbind, test.ranks)
test.ranks.matrix <- cbind(test.paths, test.ranks.matrix, "TESTING")
test.ranks.df <- data.frame(test.ranks.matrix, stringsAsFactors = FALSE)
names(test.ranks.df) <- c("Message","Date","From","Subj","Rank","Type")
test.ranks.df$Rank <- as.numeric(test.ranks.df$Rank)
test.ranks.df$Priority <- ifelse(test.ranks.df$Rank >= priority.threshold, 1, 0)

# Finally, we combine the data sets.
final.df <- rbind(train.ranks.df, test.ranks.df)
final.df$Date <- date.converter(final.df$Date, pattern1, pattern2)
final.df <- final.df[rev(with(final.df, order(Date))), ]

testing.plot <- ggplot(subset(final.df, Type == "TRAINING"), aes(x = Rank)) +
  stat_density(aes(fill = Type, alpha = 0.65)) +
  stat_density(data = subset(final.df, Type == "TESTING"),
              aes(fill = Type, alpha = 0.65)) +
  geom_vline(xintercept = priority.threshold, linetype = 2) +
  scale_alpha(guide = "none") +
  scale_fill_manual(values = c("TRAINING" = "darkred", "TESTING" = "darkblue")) +
  theme_bw()
```

备注：转移自新浪博客，截至2021年11月，原阅读数138，评论0个。
