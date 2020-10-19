---
title: 博客主题切换 TeXt -> icarus
key: icarus
tags:
- Original
toc: true
---

这两天将博客风格从 [TeXt](https://github.com/kitian616/jekyll-TeXt-theme) 切换到了 [icarus](https://github.com/ppoffice/hexo-theme-icarus)。切换的原因主要是因为 TeXt 的作者有很长一段时间没有更新了，所以最新的 release 版本存在的 bug 一直无法得到解决。

同时。随着主题更换，博客框架也从 [Jekyll](https://jekyllrb.com/) 切换到了 [Hexo](https://hexo.io/)。

<!--more-->

## Hexo 使用

Hexo 是基于 Node.js 的博客框架，其[官方文档](https://hexo.io/zh-cn/docs/)整体感觉比 Jekyll 要友好许多，对中文也有很好的支持。

[Hexo 命令](https://hexo.io/zh-cn/docs/commands) 也都比较简单好用，这里可能会用得比较多的有：

- `hexo clean`：清除缓存和已生成的静态文件
- `hexo generate`：生成静态文件
- `hexo server`：启动本地服务器，本地端口为4000
- `hexo deploy`：部署网站


## icarus 主题

[icarus 主题](https://github.com/ppoffice/hexo-theme-icarus) 是我在 Github 上搜索后发现星标比较多的一个主题，整体感觉非常清爽，同时能够展示的信息也应有尽有。icarus 的文档是通过其[示例页面](https://blog.zhangruipeng.me/hexo-theme-icarus/)以博客的形式给出的，内容也十分详细，且同时包含了中英文。

这里仅仅记录一些我认为比较有用的配置项：

### 代码折叠

在 markdown 文件中，使用如下语法来折叠代码块：

```
{% codeblock "可选文件名" lang:代码语言 >folded %}
...code block content...
{% endcodeblock %}
```

### 侧边栏位置固定

在 `_config.icarus.yml` 中配置：

```yml
sidebar:
  left:
    sticky: false
  right:
    sticky: true
```

此配置会对全局生效，如果仅仅想对具体文章的页面配置，需要创建 `_config.post.yml` 文件并在其中配置，一般来说会将目录项所在侧边栏固定。

### 博客文章显示 toc

需要在每篇博客的 [format-matter](https://hexo.io/docs/front-matter.html) 中添加 `toc: true`

### 数学公式显示

需要在 `_config.icarus.yml` 中开启插件：

```yaml
plugins:
  mathjax: true
```

### 页面布局宽度配置

- 页面宽度定义样式文件：`<icarus_directory>/include/style/responsive.styl`
- 挂件定义文件：`<icarus_directory>/layout/common/widgets.jsx`
- 主内容定义文件：`<icarus_directory>/layout/layout.jsx`

通过修改 CSS 类名中的数字改变挂件或主内容占据的栏数。数字后的屏幕尺寸，如`tablet`和`widescreen`，指代着栏数量生效的屏幕尺寸条件。

修改类名中的数字使主内容栏的栏数量和所有挂件栏的栏数量在相同屏幕尺寸下相加等于12。

## 从 Jekyll 切换到 Hexo

这里在 Hexo 文档中也有描述，需要将原 `_post` 目录中的所有文件复制到 `source/_posts` 目录中，并且在 `_config.yml` 中修改 `new_post_name` 参数：

```yml
new_post_name: :year-:month-:day-:title.md
```

在这之后还需要将原博客中存储的图片复制到 `source` 目录下，同时注意修改博客内容中对图片的引用路径。

随着主题的切换，在原先主题的 markdown 文件中的一些语法可能在新的主题下并不生效（比如原来主题对 emoji 的支持），需要将博客内容再做一遍检查。

## 博客部署

Github Page 不提供对 Hexo 的原生支持，因此我们要手动将 hexo 生成的静态页面部署到 Github 仓库中。这里我尝试使用了 [Github Actions](https://docs.github.com/en/free-pro-team@latest/actions) 提供的自动部署功能，但是并没有生效... 所以目前还是用的手动部署的方法。

博客源代码我放在 `source` 分支下，生成的静态页面放在 `master` 分支。

这里需要安装 [hexo-deployer-git](https://docs.github.com/en/free-pro-team@latest/actions)

```bash
$ npm install hexo-deployer-git --save
```

同时修改 `_config.yml` 中内容：

```yaml
deploy:
  type: git
  repo: <repository url>
  branch: master
```

之后每次更新内容后，通过 `hexo clean && hexo deploy` 就可以将最近静态文件部署到 `master` 分支。

关于 Github Action 以及目前没能生效的原因，如果之后有时间的话会研究一下~
