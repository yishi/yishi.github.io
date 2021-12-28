---
title: 购物篮分析 Market basket analysis
author: ''
date: '2015-07-20'
slug: 购物篮分析-market-basket-analysis
categories: []
tags: []
---

购物篮分析

Market basket analysis

大部分代码都在做数据清理的工作，以及整理成模型所需的格式，真正模型的代码仅一两句。

Most of the code is cleaning the data, and rearrange the data to the type of which model needed.
Codes for model only one or two lines.

注：数据以数据框格式读入时，字符型默认会保存为因子(`factor`)，字符拆分函数仅适用于字符型，而不是因子型（表面是字符，实际以数字保存），屡屡报错。需要使用`as.character`去强制转换类型，然后再进行字符拆分，或者读入数据框时，添加`stringsAsFactors = FALSE`。

PS： when you read the data as `data.frame`, the columns include strings always was treated as a factor by default, then the function `strsplit` will always give an error, because `strsplit` only use for the character, not the factor. Factor actually was number, not character.
Suggestion: use `as.character` to transform the factor to a string, or when you read the data as data frame, you should add arguments `stringsAsFactors = FALSE`.

```{r}
library(arules)
library(arulesViz)

## raw data
rule_data <- read.csv("data.csv")

## new data
rule_data2 <- data.frame(rule_data[, "清单"])
names(rule_data2) <- "items"
rule_data2$new_items <- as.character(rule_data2$items)
rule_data2$id <- 1 : nrow(rule_data2)

###  找出领用多项物资的行号
multi_items <- grep("^:", rule_data2$new_items, fixed = TRUE)
###  把一个字符串拆分成多个物资
temp <- strsplit(rule_data2$new_items[multi_items], "^:", fixed = TRUE)
###  delete 逗号后面的规格型号等附加信息
temp2 <- lapply(temp, function(y) {
  unlist(lapply(strsplit(y, "，", fixed = TRUE), function(x) {x[1]}))
})
### delete 重复的物资
temp3 <- lapply(temp2, function(x) {unique(x)})
len <- unlist(lapply(temp3, function(x) length(x)))
rep_id <- rep(multi_items, times = len)
df <- data.frame(id = rep_id, item = unlist(temp3), stringsAsFactors = FALSE)


df2 <- rule_data2[-multi_items, c("id", "new_items")]
###  delete 单个物资逗号后面的规格型号等附加信息
temp <- unlist(lapply(df2$new_items, function(y) {
  unlist(lapply(strsplit(y, "，", fixed = TRUE), function(x) {x[1]}))
}))

temp2 <- unlist(lapply(temp, function(y) {
  unlist(lapply(strsplit(y, ",", fixed = TRUE), function(x) {x[1]}))
}))

df2$item <- unlist(lapply(temp2, function(y) {
  unlist(lapply(strsplit(y, " ", fixed = TRUE), function(x) {x[1]}))
}))

### renewed data
df3 <- rbind(df, df2[, c("id", "item")])

###  转换为关联规则适用的格式
test_data <- as(split(df3[ , "item"], df3[ , "id"]), "transactions")
inspect(test_data)
```

可以使用R包自带的数据Groceries作为测试数据。

You could use data Groceries which belong to R package to test the model.

假设有10000个人购买了产品，其中购买A产品的人是1000个，购买B产品的人是2000个，
AB同时购买的人是800个。

支持度指的是关联的产品（假定A产品和B产品关联）同时购买的人数占总人数的比例，
即800/10000=8%，有8%的用户同时购买了A和B两个产品；

可信度指的是在购买了一个产品之后购买另外一个产品的可能性，例如购买了A产品之后
购买B产品的可信度=800/1000=80%，即80%的用户在购买了A产品之后会购买B产品；

提升度就是在购买A产品这个条件下购买B产品的可能性与没有这个条件下购买B产品的可能
性之比，没有任何条件下购买B产品可能性=2000/10000=20%，那么提升度=80%/20%=4。

```{r}
###  run model
rules = apriori(test_data, parameter = list(support = 0.01,confidence = 0.2))
# 按支持度查看规则
inspect(sort(rules, by = "support"))
#按置信度查看规则
inspect(sort(rules, by = "confidence"))
#也可以用subset做规则的筛选,取"右手边"含有whole milk且lift大于1.2的规则
sub.rules <- subset(rules, subset = rhs %in% "单头螺栓" & lift > 1.2)
inspect(sort(sub.rules, by = "confidence"))
#数据画频繁项的图
itemFrequencyPlot(test_data,support = 0.01, cex.names = 0.8)

plot(rules, shading = "order", control = list(main = "Two-key plot"))
plot(rules, method = "grouped")
plot(rules, method = "graph")
```

**参考资料：**
+ http://www.douban.com/note/276365088/  
+ http://cos.name/2013/02/association-rules-with-r-and-sas/  

备注：转移自新浪博客，截至2021年11月，原阅读数395，评论0个。 