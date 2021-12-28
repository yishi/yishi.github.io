---
title: 怎样把网页数据读入R
author: ''
date: '2013-09-09'
slug: 怎样把网页数据读入r
categories:
  - R
tags:
  - tips
---

以前就看了很多大牛读取网页数据，进行分析，一直想尝试，总是懒啊，各种借口，拖延到现在。

现在老板给的任务里涉及到读取api数据了，刚好学习下，有压力才有动力啊。

网络上搜集到读取的方法有以下三种：

1. 读取网页上的表格数据
来源于http://bhoom.wordpress.com/ 
```{r}
library(XML)

# URL for the Google Data
u="http://www.google.com/adplanner/static/top1000/"
tables = readHTMLTable(u)
my.table=tables[[2]]
```

2. 用R自带的函数读取网页
这里我只走了一半的路程，因为读取到的是网页的源代码，中文都成了十六进制的，英文正常，不知怎么转换继续转换成中文，遂放弃。
```{r}
read.table("http://www.baidu.com/", stringsAsFactors=FALSE)
```

3. RCurl包的应用
参考自文章：R不务正业之RCurl http://cos.name/cn/topic/17816
```{r}
library(RCurl)

myHttpheader<- c(
 "User-Agent"="Mozilla/5.0(Windows; U; Windows NT 5.1; zh-CN; rv:1.9.1.6)",
 "Accept"="text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8",
 "Accept-Language"="en-us",
 "Connection"="keep-alive",
 "Accept-Charset"="GB2312,utf-8;q=0.7,*;q=0.7"
)

d =debugGatherer()
temp<- getURL(u,httpheader=myHttpheader,
  debugfunction=d$update,verbose= TRUE)
```
第三种方法最适用于我的数据，所以，探索到此为止。
 
4. 后来发现RCurl包中有个更直接的方法
```{r}
library(RCurl)

temp<- getURI(u)
```
就OK了，不用搞什么头文件。呵呵，不过了解多些，总是好的 。
 
另外：如果你想分析的网页中有中文，导入R后可能有各种乱码出现，这时，如果你改用linux上的R，乱码就都消失了，utf8果然很强悍，但是windows下不行，有各种各样的编码方式在考验你的耐心。

5. 又發現幾個好用的，讀取網頁數據，`readLines()``download.file()`,其實這兩個就可以實現基本功能，前者直接讀取網頁的源代碼，後者可以下載動態網頁的內容，再用前者讀取每一個的源代碼。而且中文也沒有問題，你可以在參數中設置`encoding= “某種編碼方式（可以從源代碼中獲得）”`。

备注：转移自新浪博客，截至2021年11月，原阅读数391，评论0个。


