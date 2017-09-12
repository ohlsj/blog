---
title: 使用github pages和hexo搭建博客
date: 2017-09-11 21:15:27
tags:
- hexo
- github
---
参考文章:

- [hexo官方文档](https://hexo.io/zh-cn/docs/)
- [hexo主题hexo-theme-next](https://github.com/iissnan/hexo-theme-next)
- [github pages](https://pages.github.com/)

-------
基于github pages和hexo可迅速搭建个人blog，省去了购买、配置服务器等过程。虽然是静态页面，但基本能满足个人blog的需求。

<!-- more -->

#  环境

- npm
- git

#  安装

## 安装hexo

```
npm install hexo-cli -g
```

## 建站

```
hexo init blog
cd blog
npm install
```

其中`blog`为任意的目录名。

## 安装主题

`hexo`自带的主题为`landscape`，除此之外还可以使用大量的第三方主题。这里使用的主题为`next`。主题保存在`hexo`项目中的`themes`目录中。

```
mkdir themes/next
curl -s https://api.github.com/repos/iissnan/hexo-theme-next/releases/latest | grep tarball_url | cut -d '"' -f 4 | wget -i - -O- | tar -zx -C themes/next --strip-components=1
```

## 运行

`hexo server`

# hexo项目目录结构

```
.
├── _config.yml
├── package.json
├── scaffolds
├── source
|   ├── _drafts
|   └── _posts
└── themes
```
目录结构说明：

- `_config.yml`: 项目的配置文件
- `scaffolds`: 模板文件夹
- `source`: 文章和页面
- `theme`: 主题

# 配置

`hexo`的配置分为两种
- `hexo`配置文件: 位于`hexo`项目根目录中
- 主题配置文件: 位于主题目录中，例如`themes/next/_config.yml`

> 注意: 配置项中的冒号后面要有空格，否则会报错。

## 基本配置

配置文件`_config.yml`中的基本配置项包括:
```
title: ohlsj的博客
description: ohlsj的博客
author: ohlsj
language: zh-Hans
timezone: Asia/Shanghai
```

前三项分别是标题、描述和作者。

第四项可选的语言参考[hexo语言](http://theme-next.iissnan.com/getting-started.html#select-language)。

第五项时区参考[时区列表](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)

## 配置主题

首先在项目的配置文件中设置主题

```
theme: next
```

然后修改主题的配置文件，这里是配置菜单

```
menu:
  home: /
  tags: /tags
  about: /about
```

## 其他配置

### 创建`tags`和`about`页面

刚才配置好了菜单，却发现`tags`页面和`about`页面都是404。原因是`next`主题默认没有开启这些页面，需要手动开启。

首先创建两个页面

```
hexo new page "about"
hexo new page "tags"
```

其中`tags`需要配置页面类型，修改`source/tags/index.md`

```
 title: tags
 date: 2014-12-22 12:39:04
 type: "tags"
 comments: false
```

# github pages

## 配置github

新建一个名为`[username].github.io`的public项目，这里为`ohlsj.github.io`。

## 配置hexo

在`hexo`项目配置文件中：

```
deploy:
  type: git
  repo: git@github.com:ohlsj/ohlsj.github.io.git
  branch: master
```

# 部署项目

> 需要先配置好github的ssh key

```
hexo clean
hexo g
hexo d
```

此时访问`ohlsj.github.io`即可访问刚刚部署的页面。
