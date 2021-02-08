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
date: 2020-09-03 08:32:26
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
我用了最简单的那种方式：在`/public`下加一个`.html`文件。

### 一个小技巧
将需要放在`/public`的文件放在`/source`下面，这样每次`hexo d`，这些文件都会自动复制到`/public`下面。
+ `CNAME` github自定义域名所需文件
+ `googlexxxx.html` google域名验证文件

## 从github pages转到腾讯云服务器
由于某些不可描述的原因，github pages被墙了，正逢腾讯云做活动，买了个100/年的小服务器做blog。

文章没了（感谢zjh的反馈）  
~~[通过Git钩子将hexo博客自动部署到ubuntu服务器](http://www.buhuo996.com/posts/65504/)：这篇文章写的很好，设置成功后，只需要`hexo d`就可以将博客自动部署到服务器上。在这里我补充一点我遇到的问题：~~ 
```bash
git clone root@server_ip:/var/repo/hexo_static.git
```
这一步中会提示没有权限，但是我明明已经给服务器添加了密钥对，并将私钥保存到本地了啊，这是为什么呢？
在我的windows电脑上.ssh/config是这样的：
```
Host blog
    HostName 106.52.xxx.xxx
    User ubuntu
    IdentityFile C:\\Users\\Jason\\.ssh\\blog.pem
```
而且我用这条命令也是可以正常登录服务器的
```bash
ssh -o "IdentitiesOnly=yes" -i C:\\Users\\Jason\\.ssh\\blog.pem ubuntu@106.52.xxx.xxx
```
事实上这条命令与
```bash
ssh blog
```
是等价的，`Host`相当于给这个ssh配置起了一个别名

所以在`_config.yml`中

然而我发现备案太麻烦了，而且不能开留言功能，先放着吧呜呜。

## `hexo d` 某些时候不好用的原因
明明我变更了文章内容，但是`hexo d`有时候并不会将它推送上去，目前推测到的原因是hexo不会识别某篇文章内部的变更，它只会跟踪有没有新增/减少文章。

这时候我们就需要`hexo clean`, `hexo g`, `hexo d`。
