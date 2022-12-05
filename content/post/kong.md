---
title: "Netlify部署静态网站"
date: 2022-10-22
draft: false
categories: ["Netlify"]
tags: ["Netlify"]
keywords: ["Netlify","静态网站"]
---

**关于Netlify**

在它的主页上可以看到它的宣传语：
>用最快的方式构建最快的网站。

[__官方文档__](https://docs.netlify.com/)

**为什么选择Netlify**

> 内置 CI/CD 支持自动构建拉取代码仓库，每次提交的自动构建并发布预览

> 能够托管服务免费 CDN

> 能够绑定自定义域名

> 能够启用免费的TLS证书启用HTTPS

> 提供 Webhooks 和 API

> 通过内置应用程序添加动态功能


***部署一个静态网站就非常简单，此处以GitHub仓库为例***

1. 静态网站push到GitHub仓库

2. 进入[netlify](https://app.netlify.com)

3. 登录 直接使用GitHub账号，授权

4. 创建网站 点击New site from Git

  * 1. 选择仓库

  * 2. 选择分支（Branch to deploy）

  * 4.3 打包命令 （Build command）诸如 npm run build，gulp build 之类；如果本身已是静态文件，不需打包编译，这一栏则不填

  * 4.4 打包后目录 （Publish directory）诸如 public,dist，_site 之类；如果本身已是静态文件，这一栏则不填

  * 4.5 deploy site

5. 设置域名 netlify会随机生成了一个netlify下的域名，我们可以更改其前缀，并绑定到我们自己的域名下：
Site settings->Change site name 更换默认域名
Domain settings->Add custom domain添加域名，然后到域名服务商网站绑定的域名下添加一条CNAME解析，解析的主机记录即对应的netlify域名值（即 xx.netlify.com）

关于.netlify.toml自动部署参阅文档 https://docs.netlify.com/configure-builds/file-based-configuration/

更多操作后续更新。。。
