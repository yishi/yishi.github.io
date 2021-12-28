---
title: 关于批量处理excel文件
author: ''
date: '2015-11-13'
slug: 关于批量处理excel文件
categories: []
tags: []
---

**一、R的处理方法**

+ 首先，读取文件。

读取csv最方便了，速度还快，read.csv()。

但是，当你必须读取excel可使用包xlsx，该包的具体介绍参见某大神的文章http://yixuan.cos.name/cn/2012/01/new-method-to-read-excel-file-in-r/ 

+ 其次，批量处理。

自动读取某文件夹下的文件，R自带的base包就包含很多文件操作的函数。

比如这里需要用到的——list.files()，列出某文件夹下的所有文件，还可以用正则表达式找出你要的文件。


**二、python的解决之道**

因为xlsx包需要先装rJava包，这个包装的总是各种失败，因此我改用python来处理。

正好最近学习python中，在使用中学习，效果最好。

+ python 代码：
```{r}
#####  批量处理excel文件
import pandas as pd
import os

# 整理成一个函数
def foo(dir_str):
   ##  读取出某文件夹下的所有文件名
    file_name = os.listdir(dir_str)

   ##  得到所有文件的路径
    file_dir = [os.path.join(dir_str, x) for x in file_name]

    ## 读取每一个文件的某个需要计算的列，计算该列的发生次数
    single_out = [pd.ExcelFile(dir).parse(skiprows = 1, parse_cols = "I")[u'列名'].value_counts() for dir in file_dir]

    ## 把多个文件的结果合并在一起
    out = single_out[0]
    for x in single_out[1:]:
       out = out.add(x, fill_value = 0) 

    ## 导出为excel
   pd.DataFrame(out).to_excel(excel_writer = dir_str + "_out.xlsx")

#  调用函数
foo(dir_str = '某个包含所有待处理文件的路径')
```

+ ps: 

有关python中文件操作函数的介绍，

请参见博文：http://www.cnblogs.com/rollenholt/archive/2012/04/23/2466179.html 

备注：转移自新浪博客，截至2021年11月，原阅读数964，评论0个。 