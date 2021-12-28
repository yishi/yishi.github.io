---
title: 用blogdown和gitalk搭建个人网站
author: beibei
date: '2019-03-07'
slug: 用blogdown和gitalk搭建个人网站
categories:
  - R
tags:
  - blogdown
  - gitalk
---

本静态网站主要是使用R包blogdown+hugo+netlify+github搭建的，网站的评论功能使用gitalk。

只需要熟悉R，注册有github账号，对自己有充分的信心，勇于**试错**即可。

blogdown已经把hugo功能包含在内，netlify可以直接使用github账号登陆，然后进行一些参数设置。

1. 参照统计之都的[文章](https://cosx.org/2018/01/build-blog-with-blogdown-hugo-netlify-github/)搭建网站；

2. 按照上面文章操作后，出现错误信息*Site deploy failed，Error running command: Build script returned non-zero exit code: 255*，要在netlify的Advanced Build Settings 里面改，key那里填写HUGO_VERSION，version那里写你hugo的版本，比如我是0.42.2。获得hugo版本号，需要运行下面R代码
```{r}
library(blogdown)
hugo_version()
```

3. 添加网站的评论功能。参考[gitalk的官方介绍](https://github.com/gitalk/gitalk/blob/master/readme-cn.md)和[网友的搭建介绍](https://0xc000005.github.io/2017/12/19/%E4%B8%BA%E5%8D%9A%E5%AE%A2%E6%B7%BB%E5%8A%A0-Gitalk-%E8%AF%84%E8%AE%BA%E6%8F%92%E4%BB%B6/)。
具体的做法很简单，添加下面的代码到文件中，以我的目录为例"/themes/hugo-lithium/layouts/_default/single.html"
```{r, eval=FALSE}
  <!-- Link Gitalk 的支持文件  -->
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/gitalk@1/dist/gitalk.css">
  <script src="https://cdn.jsdelivr.net/npm/gitalk@1/dist/gitalk.min.js"></script>
<div id="gitalk-container"></div>     
<script type="text/javascript">
    const gitalk = new Gitalk({
    // gitalk的主要参数
		clientID: 'Github Application clientID',
		clientSecret: 'Github Application clientSecret',
		repo: 'github仓库名',
		owner: 'github用户名',
		admin: ['github用户名'],
		id:window.location.pathname,// Ensure uniqueness and length less than 50
    distractionFreeMode: true  // Facebook-like distraction free mode
    
    });
    gitalk.render('gitalk-container');
</script> 
<!-- Gitalk end -->
```

搭建完成！