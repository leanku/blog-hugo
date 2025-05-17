---
title: "GitHub 项目版本管理规范"
date: 2025-04-11T20:46:01+08:00
draft: false
categories: ["git"]
tags: ["Git"]
keywords: ["Git"]
---

# GitHub 项目版本管理规范
Git 标签（Tags）和 发布（Releases）
    在 GitHub 项目中定义和管理版本，通常结合 Git 标签（Tags）、发布（Releases） 和 语义化版本（SemVer） 来实现。以下是具体方法和最佳实践：

## 1. 使用 Git 标签（Tags）定义版本

### （1）创建版本标签

```bash
# 创建轻量标签
git tag v1.0.0

# 创建附注标签（推荐，包含提交信息和签名）
git tag -a v1.0.0 -m "Release version 1.0.0"

# 推送标签到 GitHub
git push origin v1.0.0
```

### （2）语义化版本（SemVer）
版本号格式：MAJOR.MINOR.PATCH
* MAJOR：不兼容的 API 变更
* MINOR：向后兼容的功能新增
* PATCH：向后兼容的问题修复

示例：
* v1.0.0：初始稳定版本
* v1.2.3：第 1 个大版本，第 2 次功能更新，第 3 次补丁


## 2. 通过 GitHub Releases 管理版本
### （1）手动创建 Release
1. 进入 GitHub 仓库 → Releases → Draft a new release
2. 填写：
   
   Tag version: v1.0.0（需提前创建或在此处新建）

    Release title: Version 1.0.0

    Description: 版本变更内容（可粘贴 CHANGELOG.md 对应部分）

    附件: 上传编译后的二进制文件（如 app-v1.0.0.zip）

### （2）自动生成 Release
结合 CI/CD 工具（如 GitHub Actions）自动发布：
```
# .github/workflows/release.yml
name: Release
on:
  push:
    tags:
      - 'v*'  # 监听 v 开头的标签

jobs:
  build-and-release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: make build  # 构建项目
      - uses: softprops/action-gh-release@v1
        with:
          files: |
            build/app.zip
          body: ${{ github.event.head_commit.message }}
```

## 3. 版本管理工具推荐
### （1）常用工具
|工具|	语言|	功能|安装|
|:--|:--|:--|:--|
|standard-version|	Node	|版本+CHANGELOG	|npm i -g standard-version|
|bump2version	|Python	|版本号管理	|pip install bump2version|
|git--chglog|	Go	|CHANGELOG生成|	go install github.com/git-chglog/git-chglog/cmd/git-chglog@latest

### (2) standard-version 工作流
1. 配置 package.json:
   ```
      {
      "scripts": {
        "release": "standard-version"
      }
    }
    ```
2. 提交规范：
   ```
    git commit -m "feat: 新增用户模块"
    git commit -m "fix: 修复登录异常"
   ```
3. 发布版本：
    ```
    npm run release -- --release-as minor
    git push --follow-tags
    ```

## 4. 版本分支策略 (Git Flow)
|分支|	用途|	版本示例|
|:--|:--|:--|
|main|	生产代码|	v1.0.0|
|develop|	开发版本|	v1.1.0-alpha|
|feature/*|	新功能开发|	-
|hotfix/*	|紧急修复	|v1.0.1

## 5. 版本件规范
### 1. 版本声明文件
VERSION 文件示例：
```
1.0.0
```
### 2. CHANGELOG 规范
CHANGELOG.md 示例：
```
# Changelog

## [Unreleased]
### Added
- 新增用户注册功能

## [1.0.0] - 2023-10-01
### Fixed
- 修复登录超时问题
```

## 6. 最佳实践
### 1. 版本控制原则
* 每次生产发布必须打标签
* 禁止直接修改已发布版本的代码
* 版本号只能递增，不能回退
### 2. 自动化建议
* 配置 CI 自动校验版本号格式
* 版本发布后自动生成文档
* 通过 GitHub Actions 自动发布 npm/pypi 包
### 3. 文档要求
* 每个 Release 必须包含更新说明
* 重大变更需提供迁移指南
* 维护版本兼容性矩阵
  