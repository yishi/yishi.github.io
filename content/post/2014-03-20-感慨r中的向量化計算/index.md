---
title: 感慨R中的向量化計算
author: ''
date: '2014-03-20'
slug: 感慨r中的向量化計算
categories:
  - R
tags:
  - tips
---

今天看到一篇博文，統計之都推薦的，網址見下：http://alyssafrazee.com/vectorization.html ,看了真是驚出一身冷汗，我用的特別順手的`apply`及他的兄弟姐妹（`lapply` `sapply`)等居然都是隱形循環,**根本不是向量化**,是簡潔版的循環loop,以後要用`ddply`之類的代替了。

好吧，矩陣向量化計算,請使用下面的吧。
`rowSums`, `colSums`, `rowMeans`, `colMeans` (in base), `rowSds`, `colSds`, `rowVars`, `colVars`, `rowttests`, `rowFtests` (in genefilter), `colMedians`, `rowMedians` (in matrixStats). 

告別`apply(matrix,2,sum)`之類的語句吧，隨著數據量的增加，運行時間指數增長啊，是真循環啊。

貌似`apply`是真循環，是一本庫存已久的書R-inferno（原名為R地獄，也有人翻譯為R神曲，呵呵）裏面介紹的，我卻一直沒看，傷心，要抽時間看下這本書啊。

博主原文附後：

**let's talk about vectorization** http://alyssafrazee.com/vectorization.html 
Sun 09 February 2014 | -- (permalink)

+ what's vectorization?

In R, a "vector" refers to a one-dimensional array. A "vectorized" function f() takes a vector [x1, x2, ... , xn] as input and returns the vector [f(x1), f(x2), f(x3), ... , f(xn)]. If you'd like to read more about this, you should read Chapter 3 of the R Inferno, where the third circle of R hell is "failing to vectorize."1

+ why is vectorization important?

R takes a fair amount of heat from the hacker community because it's kind of slow at looping2. It compensates (somewhat) for this weakness by using vectorized functions! Vectorized functions usually involve a behind-the-scenes loop in a low-level language (C or Fortran), which runs way faster than a pure R loop. Here's an example using the log() function that illustrates the speedup you can get by exploiting the fact that log() is vectorized:

```{r}
# illustrating log's behavior: single values and vectors
log(3)
# [1] 1.098612
log(1:10)
# [1] 0.0000000 0.6931472 1.0986123 1.3862944 1.6094379 1.7917595 1.9459101 
#[8] 2.0794415 2.1972246 2.3025851
# a vector of 1 million random numbers between 1 and 10
nums = sample(1:10, size=1000000, replace=TRUE)
# my function to call log on each vector element separately:
log_novec = function(n){
  ret = rep(NA, length(n))
  for(i in seq_along(n)){
    ret[i] = log(n[i])
    }
  return(ret)
  }
# timing results:
system.time(log_novec(nums))
# user system elapsed 
# 1.938 0.004 1.941 
> system.time(log(nums)) 
# user system elapsed 
# 0.017 0.002 0.019 
#  100x speedup!
 ```
So in conclusion: 

**vectorization is important because it allows you to operate on vectors quickly (unlike looping)**.

+ side note: a small clarification

If you are acting as an R user, you don't need to worry about WRITING vectorized functions. But if you'd like to write R code that will run in a reasonable amount of time on large dataset (memory issues aside), it's good to think about USING vectorized functions, which consequently means learning where and when to look for them. If, on the other hand, you are acting as an R developer, you might have to implement your own vectorized function, which will probably involve writing some C or Fortan.

+ how about those "apply" functions?

The apply family of functions provides some fairly clean syntax for apply-ing a function (any function) to each element of some data structure. The point I want to emphasize is this: **apply functions are basically equivalent to loops in terms of speed**. As the R Inferno puts it: apply "is not vectorization, it is loop-hiding." So using apply instead of loops might make for nicer/shorter code, but it won't make for faster code.
```{r}
system.time(sapply(nums, log)) 
# user system elapsed 
# 1.847 0.025 1.871 
system.time(vapply(nums, log, 0)
# user system elapsed 
# 0.781 0.001 0.781
 ```
(I did get a 2x speedup by using vapply and telling it that I knew the result would be numeric, but that's nothing compared to my 100x speedup from using vectorized log.)

If you are working with data frames and find yourself using lots of apply statements, check out plyr, which provides some really nice syntax, and dplyr, a superfast "next iteration" of plyr.

+ lesson 1: think twice about every loop or apply statement you write

This is what I've been doing recently: every time I write a loop or apply statement3, I think about whether I can do better. Am I calling a vectorized function inside that loop? Most functions that take a number or a string as input are vectorized. Can I somehow incorporate this into my loop code?

+ lesson 2: matrices sometimes behave like vectors

I've learned this lesson pretty recently. There are two ways in which matrices sometimes act like vectors:

   + way 1: rows (or columns) are the vector entries
   
Conceptualizing a matrix as a one-dimensional array of rows (or columns) lets us add a bunch of vectorized matrix functions to our quiver! The following vectorized matrix functions exist and are super fast and awesome: rowSums, colSums, rowMeans, colMeans (in base), rowSds, colSds, rowVars, colVars, rowttests, rowFtests (in genefilter), colMedians, rowMedians (in matrixStats). Because I think any serious R programmer should know about and use these functions whenever possible, I was surprised to see this around the Twitterverse the other day:

because, on a 10,000 x 10,000 matrix:
```{r}
mat = matrix(sample(1:10, size=100000000, replace=TRUE), nrow=10000)
system.time(apply(mat, 1, sum)) 
# user system elapsed 
# 2.191 0.343 2.534 
system.time(rowSums(mat)) 
# user system elapsed 
# 0.388 0.001 0.388
```
   + way 2: matrices get converted to vectors when you call vector functions on them
   
Here's a line of code that was bottlenecking one of my functions a few weeks ago:
```{r}
counts = lapply(1:nrow(normedcounts), function(i){ 
 (lengths[i]/1000) * normedcounts[i,]})
counts = matrix(unlist(counts, use.names=FALSE), nrow=length(counts), byrow=TRUE)
```
I had a matrix, normedcounts, where each row had been normalized by a constant. I had stored that constant (times 1000) in the vector lengths. I wanted to generate the matrix of raw (un-normalized) counts. Upon seeing this code, my advisor said "wait...are you just multiplying each row of this matrix by a number? You can just use:

$$counts = lengths/1000 * normedcounts$$

This line of code is actually a little bit tricky. There are several concepts/R quirks floating around in here - I'll illustrate them with toy examples:

+ matrices get turned into vectors by column:
```{r}
mat = matrix(1:4, byrow=TRUE, nrow=2)
mat 
#[,1] [,2][1,] 1 2[2,] 3 4
as.vector(mat)
#[1] 1 3 2 4
```
+ RECYCLING (a blessing and a curse): 

Here's an illustration of recycling, just with vectors:
```{r}
v1 = c(2,3)
v2 = 1:10
v1*v2 
# [1] 2 6 6 12 10 18 14 24 18 30
v3 = c(2,3,4)
v3*v2 
# [1] 2 6 12 8 15 24 14 24 36 20
# Warning message:In v3 * v2 : longer object length is not a multiple of shorter object length
``` 
Note that recycling happens silently unless the length of the longer vector isn't a multiple of a length of the shorter vector. This is sometimes desirable (it makes it such that constant * vector multiplication behaves as expected, without warning), but it can make debugging hellish

+ if the longer "vector" was actually a matrix, a matrix gets returned:
```{r}
mat [,1] [,2]
#[1,] 1 2
#[2,] 3 4
c(100, 50, 3)*mat 
# [,1] [,2]
#[1,] 100 6
#[2,] 150 400
# Warning message:In c(100, 50, 3) * mat : longer object length is not a multiple of shorter object length
```
Now...once you know these quirks, you can exploit them to use vectorization instead of looping (as my advisor suggested)! The three bullet points above imply that a length-k vector times a k-row matrix returns another matrix, where each row of that matrix is multiplied by the corresponding number in the vector. (That's exactly what I wanted).

Here are the timings for my example scenario:
```{r}
normedcounts = matrix(runif(1000000*100, min=0, max=5), nrow=1000000)
lengths = sample(1:100, size=1000000, replace=TRUE)
system.time(lapply(1:nrow(normedcounts), function(i){
(lengths[i]/1000) * normedcounts[i,]
})) 
# user system elapsed 
# 7.213 0.323 7.535 
system.time(lengths/1000 * normedcounts) 
# user system elapsed 
# 0.342 0.225 0.567
 ```
If you want to practice the principles of lesson 2, take the super fun puzzle challenge for readers: using no loops or apply statements, given a matrix M, find the number of outliers in each row of M. An outlier for row k is defined as an entry that is more than 3 standard deviations above the mean of row k.

+ lesson 3: methods for S3/S4 class "foo" are often vectorized for class "fooList"

I've been doing a lot of work with the GenomicRanges library lately. An object of class GRanges looks like this:
```{r}
x 
#x is of class GRangesGRanges with 5 ranges and 2 metadata columns: 
# seqnames ranges strand | id transcripts | [1] 22 [16187163, 16187278] - | 15 13 [2] 22 [16189032, 16189143] - | 16 13 [3] 22 
# #[16189264, 16189411] - | 17 13 [4] 22 [16190681, 16190791] - | 18 13 [5] 22 [16192906, 16193047] - | 19 13 --- seqlengths: 1 11 12 13 # # 14 18 19 2 22 # 4 6 7 8 X NA NA NA NA NA NA NA NA NA NA NA NA NA NA
 ```
This is basically a set of intervals ("ranges"), where each range has some associated metadata.

You can extract the width of each interval with the width() function:
```{r}
width(x)
# [1] 116 112 148 111 142
```
I had written a piece of code to operate on a list of 1296 GRanges objects. This type of list is conveniently classed as a GRangesList. Because width is a GRanges function, my immediate thought was that I would need to write an lapply statement to get a list of range widths for each of the GRanges objects in my GRangesList:

```{r}
# grl is my GRangesList
system.time(lapply(grl, width))
# user system elapsed 
# 9.481 0.010 9.490
```
This is pretty slow for a list of length 1296, so I did a bit of documentation-reading and found out that the width() method is vectorized: you can call it on a GRangesList...

```{r}
system.time(width(grl)) 
# user system elapsed
# 0.001 0.000 0.001
 ```
...WITH AMAZING RESULTS. 9000X SPEEDUP, BABY.

+ lesson 4: sometimes you really do need a loop

Alas, vectorized functions can't solve all of our problems. For example, loops with dependent iterations, like Gibbs samplers and Hidden Markov Models, will probably need to stay loops. All is not lost, however: if you (or your friends!) know a lower-level language, you (all) can write the functions yourself, perhaps using a tool like Rcpp to interface with R.

+ footnotes

   + The R Inferno is hilarious and informative reading for anyone who loves and/or hates R.
   
   + R takes heat from the hacker community for a number of other reasons as well. We can (maybe) talk about this another day, but what I'll say now is this: I don't like when people that have never used R go on and on about how terrible it is. The way I feel about R is how I imagine I'd feel about my dorky little brother (if I had one): he's quirky and weird and not the best athlete ever, but you don't get to make fun of him because he's my brother, not yours. Incidentally, this is also the way I feel when people hate on Baltimore.

   + I find myself writing loops and apply statements embarrassingly often. I suspect this happens for two reasons: 1. I find loops more intuitive/expressive: they more closely match how I conceptualize the code in my head, and 2. the programming language I learned first, and the only one I ever learned in a formal classroom setting, is C++.

备注：转移自新浪博客，截至2021年11月，原阅读数101，评论0个。

