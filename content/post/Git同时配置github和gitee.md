---
title: "Git同时配置github和gitee"
date: 2023-02-26
draft: false
categories: ["git"]
tags: ["Git"]
keywords: ["Git"]
---

# Git同时配置github和gitee

## 想要同时使用GitHub和Gitee，但是每次输入密码就会很麻烦。为此，本文将介绍 Git 同时配置 **github**和 **gitee** 的ssh key

``` shell
#查看git配置
git config --global  --list
#没配置过的话先配置用户名和邮箱信息
git config --global user.name “username”
git config --global user.email “email”
#默认没有.ssh目录，
ssh-keygen -t rsa -C "xxxxxxx@xx.com"
#在windows下会生成.ssh目录 [c盘/用户/用户名/.ssh]
```

### 1.  查看ssh文件
``` shell
cd ~/.ssh**  

ls
```
### 2.  删除之前的.ssh
``` shell
 rm -rf id_rsa id_rsa.pub
```
### 3. 生成gitee和github 的 SSH Key
``` shell
ssh-keygen -t rsa -C "xxxxxxx@xx.com" -f "id_rsa_github"
ssh-keygen -t rsa -C "xxxxxxx@xx.com" -f "id_rsa_gitee"
```
### 4. 查看SSH Key
``` shell
cat id_rsa_github.pub**
```
### 5. 拷贝 ssh-rsa 开头的 ssh key，然后到github上添加ssh key

###  6. 添加 config解决ssh冲突
``` shell
vim config

# 文件类容如下
# gitee
Host gitee.com
HostName gitee.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/id_rsa_gitee
 
# github
Host github.com
HostName github.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/id_rsa_github
```
###  7. 测试是否配置成功
``` shell
ssh -T git@gitee.com  
ssh -T git@github.com

# 配置成功 正常会输出如下
Hi leanku! You've successfully authenticated, but GitHub does not provide shell access.
```