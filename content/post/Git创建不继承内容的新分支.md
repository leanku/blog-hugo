---
title: "Git创建不继承内容的新分支"
date: 2023-02-26T19:46:01+08:00
draft: false
categories: ["git"]
tags: ["Git"]
keywords: ["Git"]
---

# Git创建不继承内容的新分支方法

在 Git 中，想创建一个新的分支，并且不想要当前分支的任何内容（即希望新分支是完全干净的，不包含当前分支的修改或提交），可以按照以下步骤操作：


## 方法 1：基于远程默认分支（如 main/master）创建新分支
如果你希望新分支是基于远程仓库的默认分支（而不是当前分支），可以这样做：
    ```
    # 1. 确保本地没有未提交的修改（否则会提示你提交或暂存）
    git status

    # 2. 拉取远程最新代码（确保本地默认分支是最新的）
    git fetch origin

    # 3. 基于远程默认分支（如 origin/main）创建新分支
    git checkout -b 新分支名 origin/main

## 方法 2：基于某个提交或分支创建全新分支
如果你想基于某个特定的提交（而不是当前分支）创建新分支：
    ```
    # 1. 先切换到目标分支或提交（例如切换到 main 分支）
    git checkout main

    # 2. 拉取最新代码（可选）
    git pull origin main

    # 3. 创建并切换到新分支
    git checkout -b 新分支名

## 方法 3：强制创建孤立分支（完全无历史）
如果你想创建一个完全独立的新分支（不继承任何历史）：

1. 在切换分支之前，确保你没有未提交的修改，否则 Git 可能会尝试携带这些修改到新分支：
    ```
    git status

    # 提交他们
    git add .
    git commit -m "暂存当前修改"

    # 或者丢弃它们
    git reset --hard  # 强制丢弃所有未提交的修改
    ```
2. 创建一个完全独立的新分支（不继承任何历史）
    ```
    git checkout --orphan 新分支名
    ```
    这会创建一个空白分支，没有任何提交历史。之后你需要手动添加文件并提交：

3. 清理可能残留的文件（如果有）

    虽然 --orphan 会创建一个空分支，但 Git 可能会保留一些未被跟踪的文件（比如 node_modules/、build/ 等）。你可以手动清理：
    ```
    git rm -rf .  # 删除所有已跟踪的文件（如果有）

    git clean -fd  # 删除所有未被跟踪的文件和目录
    ```
4. 提交初始空状态

    如果你想保持分支干净，可以提交一个空目录：
    ```
    git commit --allow-empty -m "Initial commit (empty branch)"
    ```
    推送到远程仓库
    ```
    git push -u origin 新分支名
    ```
