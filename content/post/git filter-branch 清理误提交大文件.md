---
title: "Git创建不继承内容的新分支"
date: 2025-05-10T19:46:01+08:00
draft: false
categories: ["git"]
tags: ["Git"]
keywords: ["Git"]
---

# git filter-branch 清理误提交大文件

## 问题概述：
在当前项目中放有大文件（超过100M）package/grpc.tar.gz,进行了git add,commit 并push, push过程中出现了错误如：

    remote: error: File package/grpc.tar.gz is 973.06 MB; this exceeds GitHub's file size limit of 100.00 MB


这个问题会导致后面的提交以及push出错


## git filter-branch
1. 用 git filter-branch 删除历史中所有的 grpc.tar.gz

    ```
    git filter-branch --force --index-filter \
    "git rm --cached --ignore-unmatch package/grpc.tar.gz" \
    --prune-empty -- lite-AlmaLinux
    ```
💡 说明：这个命令会清理当前分支里所有 commit 中的 package/grpc.tar.gz

2. 清理垃圾对象并压缩仓库大小
    ```
    git reflog expire --expire=now --all
    git gc --prune=now --aggressive
    ```

3. 然后重新 push（第一次 push 分支）
    ```
    git push --set-upstream origin lite-AlmaLinux
    ```

## 防止此类问题
1. 优化 .gitignore， 比如
    ```gitignore
    # 忽略大文件
    *.tar.gz
    ```
    然后
    ```
    git add .gitignore
    git commit -m "update: ignore tar.gz files"
    git push
    ```

