---
title: "Git历史重置"
date: 2023-02-26
draft: false
categories: ["git"]
tags: ["Git"]
keywords: ["Git"]
---

# Git历史重置
有时候需要克隆github的项目使用，自己进行修改需要提交到自己的代码仓库，但是会包含前仓库的大量提交历史，看起来不方便，就需要清理历史记录并重置为一个全新的提交

## 新项目（如git clone 的项目）

``` shell
# 进入项目目录
cd template-repo-master
# 删除原有的.git目录（如果存在）初始化全新的Git仓库
rm -rf .git
git init

git add .
git commit -m "Initial commit from template"

git remote add origin git@github.com:your-username/your-new-repo.git

git push -u origin master --force
```

## 已经将代码推送到自己的远程仓库 清理历史记录并重置为一个全新的提交
``` shell
# 1. 克隆你的远程仓库（可选）
git clone git@github.com:your-username/your-repo.git
cd your-repo

# 2.创建孤立分支（全新的历史起点）
git checkout --orphan new-branch

# 3. 添加所有文件到新分支
git add -A
git commit -m "Initial commit (cleaned history)"

#4. 删除旧的主分支
git branch -D master  # 或 main

# 5. 重命名新分支为主分支
git branch -m master  # 或 main

# 6.  强制推送到远程仓库
git push -f origin master

# 7. 更新远程仓库设置（如有必要）
git branch --set-upstream-to=origin/master master

# 检查提交历史应该只显示你的"Initial commit"
git log --oneline
```