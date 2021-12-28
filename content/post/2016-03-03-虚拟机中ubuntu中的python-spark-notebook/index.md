---
title: 虚拟机中ubuntu中的python spark notebook
author: ''
date: '2016-03-03'
slug: 虚拟机中ubuntu中的python-spark-notebook
categories: []
tags: []
---

最近尝试spark，第一步就是要搭环境啊，简单记录下，方便将来参考。

装虚拟机、装ubuntu系统、装python、装spark。

**主机和虚拟机之间的文件共享。**
+ 参考网址：http://123zhoulidong.blog.163.com/blog/static/111644814201321811240203/  
+ 因为共享盘是临时加载的，所以每次用时都要运行下面命令：
+ sudo mount -t vboxsf share_file /mnt/shared

**直接打开notebook**
+ cd（转到家目录）
+ ipython notebook （启动程序）
+ ctrl+alt+F7 （切换到ubuntu的图形界面）
+ http://localhost:8888/ （打开浏览器，输入网址）

**直接运行pyspark**
+spark存贮在根目录下
+ cd /spark
+ ./bin/pyspark
+ 打开运行

**要用notebook打开pyspark**
+ 需要在notebook的目录下，即家目录/home/username
+ cd（转换到家目录下）即可， 然后运行
+ IPYTHON_OPTS="notebook" /spark/bin/pyspark
+ 然后ctrl+alt+F7到图形界面 打开火狐浏览器 输入本地目录http://localhost:8888/
+ 新建一个文件夹，然后新建一个python2文件，即为notebook，搞定。
+ 其他更详细的配置方法：http://blog.cloudera.com/blog/2014/08/how-to-use-ipython-notebook-with-apache-spark/ 

windows上尝试失败 因为cmd的命令行不识别IPYTHON_OPTS命令。

备注：转移自新浪博客，截至2021年11月，原阅读数186，评论0个。