---
title: R包data.table备忘
author: ''
date: '2017-09-08'
slug: r包data-table备忘
categories: []
tags: []
---

当你的数据集有百万条，或600MB以上时，你需要data.table包协助，加速你的操作，data.table是data.frame的加强版。

1. 读取文件，忘记~~read.csv~~， 使用<u>fread</u>吧，速度快，还有进度提醒。

2. 提取某些行的数据，忘记~~vector scan~~， 使用<u>binary search</u>吧，速度快。

~~df[df$x == "R" & df$y == "h", ])    # vector scan~~

```{r}
dt = as.data.table(df)
setkey(dt, x, y)
# binary search
dt[list("R", "h')]          
#or binary search
dt[.("R", "h')]             
```
3. 分组，并进行计算

<u>简单版本：</u>
```{r}
dt[, sum(v), by = x]  
#或者
dt[, sum(v), by = "x, y"]
#      x        V1
# 1： A       192214
# 2： B        192183
```
<u>复杂版本：</u>
```{r}
DT = as.data.table(iris)
whatToRun = quote( .(AvgWidth = mean(Sepal.Width),
                              MaxLength = max(Sepal.Length)) )
DT[, eval_r(whatToRun), by=.(FirstLetter=substring(Species,1,1))]
#    FirstLetter AvgWidth   MaxLength
# 1: s             3.428        5.8
# 2: v             2.872        7.9
```
<u>进阶版本：</u>
```{r}
DT[x>1000, sum(y*z), by=w]
```
4. 取列
```{r}
DT[, region]  
#或者
DT[, "region", with = FALSE]  或者DT[["region"]]
# 取多个列
DT[, c(1, 4, 10), with = FALSE]

DT[, region]      #返回一个向量
DT[, .(region)]   #返回一个 data.table
```

5. 查询串联在一起

These queries can be **chained** together just by adding another one on the end:
```{r}
DT[...][...]
# sum(v) by colA then return the 6 largest which are under 300
DT[, sum(v), by = colA] [V1<300] [tail(order(V1))]
```
6. 添加、删除、更新某些列：=
```{r}
DT[, V1 := round(exp(V1),2)]

DT[, c("V1","V2") := list(round(exp(V1),2), LETTERS[4:6])] 
#or
DT[, ':=' (V1 =round(exp(V1),2),V2 = LETTERS[4:6])][]

DT[, V1 := NULL] # 移除一列

DT[, c("V1","V2") := NULL] # 移除多列

Cols.chosen = c("V1","V2")

DT[, Cols.chosen := NULL]

#无显式的返回结果，列名为Cols.chosen的列将会被删除

#删除指定变量Cols.chosen包含的V1列和V2列

DT[, (Cols.chosen) := NULL]
```
7. 其他

**.N**

.N可以用来表示行的数量或者最后一行
```{r}
# 在i处使用：
DT[.N-1]
#    V1 V2      V3 V4
# 1:  1  B -0.5765 11
# 返回每一列的倒数第二行

# 在j处使用：
DT[,.N-1]
# [1] 11
# 返回倒数第二行所在的行数。
```

**by=.EACHI参数**

by=.EACHI允许按每一个已知i的子集分组，在使用by=.EACHI时需要设置键值 

返回键值(V2列)中包含A或C的所有行中，V4列的总和。

```{r}
DT[c("A","C"), sum(V4)]
# [1] 52
# 返回键值所在列(V2列)中包含A的行在V4列总和与包含C的行在V4列的总和。

DT[c("A","C"), sum(V4), by=.EACHI]
#   V2 V1
# 1: A 22
# 2: C 30
```

**.SD参数**

.SD是一个data.table，他包含了各个分组，除了by中的变量的所有元素。.SD只能在位置j中使用：

```{r}
DT[, print(.SD), by=V2]
#    V1      V3 V4
# 1:  1 -0.8313  1
# 2:  2 -0.6264  4
# 3:  1 -0.5765  7
# 4:  2  0.7615 10
#    V1      V3 V4
# 1:  2  0.7615  2
# 2:  1 -0.8313  5
# 3:  2 -0.6264  8
# 4:  1 -0.5765 11
#    V1      V3 V4
# 1:  1 -0.5765  3
# 2:  2  0.7615  6
# 3:  1 -0.8313  9
# 4:  2 -0.6264 12
# Empty data.table (0 rows) of 1 col: V2
```
以V2为分组，选择每组的第一和最后一行：
```{r}
DT[,.SD[c(1,.N)], by=V2]
#    V2 V1      V3 V4
# 1:  A  1 -0.8313  1
# 2:  A  2  0.7615 10
# 3:  B  2  0.7615  2
# 4:  B  1 -0.5765 11
# 5:  C  1 -0.5765  3
# 6:  C  2 -0.6264 12
```

以V2为分组，计算.SD中所有元素的和:
```{r}
DT[, lapply(.SD, sum), by=V2]
#    V2 V1      V3 V4
# 1:  A  6 -1.2727 22
# 2:  B  6 -1.2727 26
# 3:  C  6 -1.2727 30
```
**.SDcols**

.SDcols常于.SD用在一起，他可以指定.SD中所包含的列，也就是对.SD取子集：

```{r}
DT[, lapply(.SD,sum), by=V2,
   .SDcols = c("V3","V4")]
#    V2      V3 V4
# 1:  A -1.2727 22
# 2:  B -1.2727 26
# 3:  C -1.2727 30
```
.SDcols也可以是一个函数的返回值：

```{r}
DT[, lapply(.SD,sum), by=V2,
   .SDcols = paste0("V",3:4)]
#    V2      V3 V4
# 1:  A -1.2727 22
# 2:  B -1.2727 26
# 3:  C -1.2727 30
```
结果与上一个是相同的。

**使用set()家族**

**set()**

set()通常用来更新给定的行和列的值，要注意的是，他不能跟by结合使用。

```{r}
rows = list(3:4,5:6)
cols = 1:2
for (i in seq_along(rows))
{ 
set(DT,
i=rows[[i]],
j = cols[i],
value = NA) 
}

DT
 #    V1 V2      V3 V4
 # 1:  1  A -0.0559  1
 # 2:  2  B -0.4450  2
 # 3: NA  C  0.0697  3
 # 4: NA  A -0.1547  4
 # 5:  1 NA -0.0559  5
 # 6:  2 NA -0.4450  6
 # 7:  1  A  0.0697  7
 # 8:  2  B -0.1547  8
 ```
以上程序把给定的一组行和列都设置为了NA

**setname()**

与set()同理，setname()可以修改给定的列名和行名，以下程序是

```{r}
#把名字为"old"的列，设置为"new"
setnames(DT,"old","new") 
#把"V2","V3"列，设置为"V2.rating","V3.DataCamp"
setnames(DT,c("V2","V3"),c("V2.rating","V3.DataCamp"))
```

**setcolorder()**

setcolorder()可以用来修改列的顺序。

```{r}
setcolorder(DT,c("V2","V1","V4","V3"))
#这段代码会使得列的顺序变成：
# [1] "V2" "V1" "V4" "V3"
```

**参考**

+ data.table包帮助文件中的10分钟快速入门和FAQ文档。
+ https://github.com/Rdatatable/data.table/wiki  
+ http://www.cnblogs.com/nxld/p/6059570.html 

**推荐一个学习data.table的资源**
+ https://www.datacamp.com/courses/data-analysis-the-data-table-way  


**再推荐一本关于R的新书**   
+ R for Data Science，网页版免费阅读。http://r4ds.had.co.nz/index.html  


备注：转移自新浪博客，截至2021年11月，原阅读数34，评论0个。  