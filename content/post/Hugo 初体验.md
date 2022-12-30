---
title: "Hello Hugo"
date: 2022-10-21
draft: false
categories: ["Hugo"]
tags: ["Hugo"]
keywords: ["Hugo","静态网站"]
---

Hugo 是用 [Go](http://golang.org/) 语言写的，支持多个平台 包括  Windows,  Linux,  FreeBSD 和  OS X (Darwin) for x64, i386 和 ARM architectures.

[__官方文档__](https://gohugo.io/overview/introduction/)

**为什么选择hugo**

> 通过 Hugo 你可以快速搭建你的**静态网站**，比如博客系统、文档介绍、公司主页、产品介绍等等。相对于其他静态网站生成器来说，Hugo 具备如下特点：

> 极快的页面编译生成速度。（ ~1 ms 每页面）

> 完全跨平台支持，可以运行在  Mac OS X,  Linux,  Windows, 以及更多!

> 安装方便 [Installation](https://www.gohugo.org/doc/overview/installing/)

> 本地调试 [Usage](https://www.gohugo.org/doc/overview/usage/) 时通过 [LiveReload](https://www.gohugo.org/doc/extras/livereload/) 自动即时刷新页面。

> 完全的皮肤支持。

> 可以部署在任何的支持 HTTP 的服务器上。



## 一.安装

下载最新的 release 版本 [Hugo Releases](https://github.com/spf13/hugo/releases)， Windows,  Linux,  FreeBSD 和  OS X (Darwin) for x64, i386 和 ARM architectures.

Windows安装时需要加入环境变量

Mac就比较方便直接执行：`brew install hugo`

安装成功之后的操作如下

```bash
#查看版本
hugo version
#创建站点 如在当前目录创建名为blog的站点
hugo new site blog
cd blog
#目录如下
  ▸ archetypes/
  ▸ doc/content/
  ▸ data/
  ▸ layouts/
  ▸ static/
    config.toml
```

## 二.创建文章

```bash
#创建文章
hugo new post/first.md

#新创建的文件会在 content/post/first.md

```

## 三.安装主题

找到心仪的[__主题__](https://themes.gohugo.io/)，以PaperMod为例

到站点存放主题目录blog/theme/

 执行 git clone 或者直接下载主题解压到/theme目录下

 按自己需要修改配置文件config.tomal

## 四.运行

执行`hugo server`

浏览器访问 [__http://localhost:1313__](http://localhost:1313) 

更多操作可自行阅读文档...



