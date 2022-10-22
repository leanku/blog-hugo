---
title: "Git版本升级"
date: 2022-10-21
draft: false
categories: ["Issue"]
tags: ["Mac","Git","brew"]
keywords: ["Mac","Git","brew"]
---

使用 [Brew](https://brew.sh/) 对Mac系统Git进行版本升级


```bash
#安装最新的Git
➜  ~ brew install git
#改变默认Git指向
➜  ~ which git
#/usr/bin/git
➜  ~ git --version
#git version 2.24.1 (Apple Git-126)
➜  ~ brew link git --overwrite
#新开终端查看版本
➜  ~ git version
#git version 2.38.1
```

可以看到，我们的 git 版本已经升级到最新版了



