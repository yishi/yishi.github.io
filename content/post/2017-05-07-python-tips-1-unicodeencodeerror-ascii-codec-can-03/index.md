---
title: 'python tips 1: UnicodeEncodeError: ''ascii'' codec can&#03'
author: ''
date: '2017-05-07'
slug: python-tips-1-unicodeencodeerror-ascii-codec-can-03
categories:
  - python
tags:
  - tips
---

1. python 2.7 的UnicodeEncodeError: 'ascii' codec can't encode 异常错误

数据中有中文，输出时就遇到了这个错误。

解决方法：
```{python}
import sys
reload(sys)
sys.setdefaultencoding('utf8')
```

解决了问题1，出现了问题2.


备注：转移自新浪博客，截至2021年11月，原阅读数29，评论0个。