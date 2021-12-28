---
title: R 包 dplyr 备忘
author: ''
date: '2017-09-22'
slug: r-包-dplyr-备忘
categories: []
tags: []
---

+ 选取某些行 filter()
+ 排序 arrange()
+ 选某些列 select()
+ 根据其他列新增某列变量 mutate()
+ 汇总  summarise()
+ 上面的都可以和 group_by() 分组配合使用

```{r}
filter(flights, month == 1, day == 1)

filter(flights, month == 11 | month == 12)

arrange(flights, year, month, day)

arrange(flights, desc(arr_delay))

select(flights, year, month, day)

select(flights, -(year:day))
starts_with("abc")
ends_with("xyz")
contains("ijk")
matches("(.)\\1")
num_range("x", 1:3)

rename(flights, tail_num = tailnum)

# This is useful if you have a handful of variables you’d like to move to the start of the data frame.

select(flights, time_hour, air_time, everything())

flights_sml <- select(flights, 
  year:day, 
  ends_with("delay"), 
  distance, 
  air_time
)

mutate(flights_sml,
  gain = arr_delay - dep_delay,
  speed = distance / air_time * 60
)

# If you only want to keep the new variables, use transmute()
transmute(flights,
  dep_time,
  hour = dep_time %/% 100,
  minute = dep_time %% 100
)
```
+ Logs: log(), log2(), log10(). 

Logarithms are an incredibly useful transformation for dealing with data that ranges across multiple orders of magnitude. They also convert multiplicative relationships to additive, a feature we’ll come back to in modelling.

All else being equal, I recommend using log2() because it’s easy to interpret: a difference of 1 on the log scale corresponds to doubling on the original scale and a difference of -1 corresponds to halving.

```{r}
(x <- 1:10)
#>  [1]  1  2  3  4  5  6  7  8  9 10

lag(x)
#>  [1] NA  1  2  3  4  5  6  7  8  9

lead(x)
#>  [1]  2  3  4  5  6  7  8  9 10 NA
```

+ Arithmetic operators: +, -, *, /, ^. 

These are all vectorised, using the so called “recycling rules”. If one parameter is shorter than the other, it will be automatically extended to be the same length. This is most useful when one of the arguments is a single number: air_time / 60, hours * 60 + minute, etc.

+ Cumulative and rolling aggregates: 

R provides functions for running sums, products, mins and maxes:cumsum(), cumprod(), cummin(), cummax(); and dplyr provides cummean() for cumulative means. If you need rolling aggregates (i.e. a sum computed over a rolling window), try the RcppRoll package.

```{r}
x
#>  [1]  1  2  3  4  5  6  7  8  9 10

cumsum(x)
#>  [1]  1  3  6 10 15 21 28 36 45 55

cummean(x)
#>  [1] 1.0 1.5 2.0 2.5 3.0 3.5 4.0 4.5 5.0 5.5
```

+ Logical comparisons, <</code>, <=, >, >=, !=

+ Ranking: 

there are a number of ranking functions, but you should start with min_rank(). It does the most usual type of ranking (e.g. 1st, 2nd, 2nd, 4th). The default gives smallest values the small ranks; usedesc(x) to give the largest values the smallest ranks.

```{r}
y <- c(1, 2, 2, NA, 3, 4)
min_rank(y)
#> [1]  1  2  2 NA  4  5

min_rank(desc(y))
#> [1]  5  3  3 NA  2  1

row_number(y)
#> [1]  1  2  3 NA  4  5

dense_rank(y)
#> [1]  1  2  2 NA  3  4

percent_rank(y)
#> [1] 0.00 0.25 0.25   NA 0.75 1.00

cume_dist(y)
#> [1] 0.2 0.6 0.6  NA 0.8 1.0

# The last key verb is summarise(). It collapses a data frame to a single row:
summarise(flights, delay = mean(dep_delay, na.rm = TRUE))

delays <- flights %>%   group_by(dest) %>%   summarise(
    count = n(),
    dist = mean(distance, na.rm = TRUE),
    delay = mean(arr_delay, na.rm = TRUE)
  ) %>%   filter(count > 20, dest != "HNL")

delays %>%   
  filter(n > 25) %>%   
  ggplot(mapping = aes(x = n, y = delay)) +     

geom_point(alpha = 1/10)
```

+ Measures of location: 

we’ve used mean(x), but median(x) is also useful. The mean is the sum divided by the length; the median is a value where 50% of x is above it, and 50% is below it.
Measures of spread: sd(x), IQR(x), mad(x). The mean squared deviation, or standard deviation or sd for short, is the standard measure of spread. The interquartile range IQR() and median absolute deviation mad(x) are robust equivalents that may be more useful if you have outliers.

+ Measures of rank: 

min(x), quantile(x, 0.25), max(x). Quantiles are a generalisation of the median. For example, quantile(x, 0.25) will find a value of x that is greater than 25% of the values, and less than the remaining 75%.
```{r}
not_cancelled %>%   
  group_by(dest) %>%   
  summarise(distance_sd = sd(distance)) %>%
  arrange(desc(distance_sd))
```  
  
+ Measures of position: 

first(x), nth(x, 2), last(x). These work similarly to x[1], x[2], andx[length(x)] but let you set a default value if that position does not exist (i.e. you’re trying to get the 3rd element from a group that only has two elements). For example, we can find the first and last departure for each day:
```{r}
not_cancelled %>%   
  group_by(year, month, day) %>%   
  mutate(r = min_rank(desc(dep_time))) %>%   
  filter(r %in% range(r))
```
+ Counts: 

You’ve seen n(), which takes no arguments, and returns the size of the current group. To count the number of non-missing values, use sum(!is.na(x)). To count the number of distinct (unique) values, use n_distinct(x)
Counts are so useful that dplyr provides a simple helper if all you want is a count:
```{r}
not_cancelled %>%
  count(dest)
  daily %>%   
  ungroup() %>%             # no longer grouped by date  
  summarise(flights = n())  # all flights
```

+ 汇总下：
   + +、-、*、\、^
   + <</code>, <=, >, >=, !=
   + log()
   + lag() lead()
   + cumsum()
   + min_rank()
   + median()
   + IQR()
   + quantile()
   + nth()

**参考自：**

+ http://r4ds.had.co.nz/transform.html  

好书啊，建议从头到尾阅读一遍，查缺补漏，温故而知新，可以为师矣。


备注：转移自新浪博客，截至2021年11月，原阅读数55，评论0个。 