---
title: 博客折腾日记
top: false
cover: false
toc: true
mathjax: true
tags:
  - hexo
categories:
  - [blog]
date: 2020-07-10 08:32:26
summary:
---

# 博客折腾日记

## 主题
一次偶然的机会打开了别人的博客，它的界面很好看，于是我自己也想做一个。
他用的是`hexo+next`，我一开始用的主题是[yilia](https://github.com/litten/hexo-theme-yilia)，这个主题配置简单，就是有点丑，而且很久没更新了。
配置好后发现：文章预览有问题，主页展示了整篇文章，这个地方折腾了好久，有一天突然又能自动截断了......  

但是后来我上Github找到一个非常好看的主题[hexo-theme-matery](https://github.com/blinkfox/hexo-theme-matery)，这个的配置教程超级好，基本都覆盖到了。
有一些没覆盖到的内容可以去Issues找或者搜索代码，比如：
1. `mathjax`默认全局打开:`/layout/post.ejs`的最后一个`if(xxx)`改成`if(1)`即可

其实博客关键还是内容，内容为王，外观再漂亮也只是衬托罢了。

## 搜索引擎收录
### Google
上[Google Search Console](https://search.google.com/search-console/welcome)，有多种验证方式来确认网站是归属你所有的。  
我用了最简单的那种方式：在`/public`下加一个`.html`文件，但是每次部署网站`hexo clean` `hexo d`都要手动把这个文件加进去