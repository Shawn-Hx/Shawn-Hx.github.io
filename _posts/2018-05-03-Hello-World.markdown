---
layout: post
title:  "Hello World"
---

## 关于博客 ##

很早就有了写博客的想法，但是耽搁到大三下才写下这第一篇博客，原因大概有这么几点。

最主要的原因是**懒**。我一直是个极其懒惰的人，微博只有转发，也从不发说说和朋友圈，更别说发布和更新博客了。

不仅如此，写博客还是一件极其**麻烦**的事情。如果要写博客，我是肯定不会在诸如CSDN这样的博客网站上写的，不仅界面丑陋而且充斥着各种抄袭、误导人的文章。既然如此那就得自己建站，买域名、买服务器，加上之后的部署和维护，极其麻烦，所以也一直没有动手。

当然还有很重要的一个问题是：**博客应该写什么？**

看过[阮一峰][ruanyifeng]、[廖雪峰][liaoxuefeng]、[耗子哥][coolshell]、[垠神][wangying]等诸多dalao的博客，深感自己当前知识的匮乏、对技术理解之浅，对比之下觉得自己写出的技术博客必然会十分肤浅。而如果是记录一些踩过的坑，或是写课堂学习笔记又觉得没有太多的价值，也提不起写博客的兴趣，总之就这样一直搁置了下来。

[ruanyifeng]:	http://www.ruanyifeng.com/
[liaoxuefeng]:	https://www.liaoxuefeng.com/
[coolshell]:	https://coolshell.cn/
[wangying]:		http://www.yinwang.org/

直到前一阵子听说，[GitHub Pages](https://pages.github.com/)可以免费搭建个人博客，绑定域名，甚至能免费开启https，我这才又重新想起了写博客的事。几乎每天都在用的GitHub有这么方便的工具，我居然才知道，确实有点孤陋寡闻了。恰好最近在微信读书上看书，苦于看过的书前面看后面忘，把博客作为读书总结归纳的地方再好不过。

### 创建仓库 ###

话不多说，我先改掉了GitHub的用户名（原来的用户名有点蠢），按username.github.io的命名要求新建了仓库。配置好了[Jekyll](https://www.jekyll.com.cn/)，并学习了下基本的用法，用Jekyll创建了自己的博客并push到了我的仓库中。

#### Jekyll ####

Jekyll的安装很简单。首先需要在Mac上安装Xcode和Command-Line Tools。Xcode可以直接在App Store搜索下载，下载完成后在终端执行`xcode-select --install`后根据提示即可安装Command-Line Tools。

Mac OS X自带了Ruby，直接在终端下执行`gem install jekyll`即可完成安装。

如果系统自带的Ruby版本较低，安装时可能会提示版本较低，这时需要安装rvm，使用rvm安装和管理新版本Ruby。这个[知乎提问](https://www.zhihu.com/question/66800711)的回答讲解比较具体。

接下来直接终端执行`jekyll new myblog`，jekyll会自动创建一个内容符合博客网站目录结构的myblog目录。在生成的myblog目录下，执行`jekyll serve`，即可在本地浏览器访问`http://localhost:4000`进行预览。

如果中途遇到缺少依赖的提示，使用`gem install [package]`安装即可。

最后将myblog目录下的内容push到之前创建的github仓库中，便可以访问到博客内容。

#### 模版 ####

Jekyll有诸多主题模版，Github提供了一些可选的主题，可以在仓库设置中选择。除此之外<http://jekyllthemes.org/> 也提供了大量主题可供选用。

目前我还是使用默认的Minima主题，也许以后有空的时候会摸索一下换个好看一点的。

### 域名 ###

GitHub Pages给博客提供的默认域名即为仓库名`username.github.io`。如果需要自定义域名，可以在项目根目录下创建`CNAME`文件，输入不带协议的自定义域名即可。

我是在[GoDaddy](https://sg.godaddy.com/zh)购买的域名，原本在万网上发现.com .cn .net常用域名都已经被注册了（意料之中），后来发现.me域名不错，用在个人博客恰到好处，而万网并不提供.me域名的购买，于是最终在GoDaddy花了￥24购买了首年的huangxiao.me域名。

GitHub Pages从5月1日起可以为自定义域名支持HTTPS。新的IP地址不仅支持HTTPS，而且可以通过CDN加快访问速度，同时还能防范DDoS攻击。

- 如果自定义域名使用CNAME或ALIAS记录，那么仅需在仓库设置中勾选`Enforce HTTPS`选项即可。
- 如果自定义域名使用A记录，则需要设置A记录将自定义域名指向以下IP：
	- 185.199.108.153
	- 185.199.109.153
	- 185.199.110.153
	- 185.199.111.153

[具体教程链接](https://help.github.com/articles/setting-up-an-apex-domain/)

## Hello World ##

于是便有了这一篇博客，以上记录的过程前前后后经过了几个星期的时间。因为太久不写文章，所以语言组织和表达能力还需要多加练习。

无论如何，总算是有了一个方便的写博客的地方，也有了第一篇博客文章。希望自己能够不断提高，写出和上面提到的dalao们一样高质量的博客文章。
