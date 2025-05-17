---
title: "Jenkins 使用 "
date: 2025-05-09T20:46:01+08:00
draft: false
categories: ["Jenkins "]
tags: ["Jenkins "]
keywords: ["Jenkins "]
---

# Jenkins 使用

## 1. [安装Jenkins](https://www.jenkins.io/doc/book/installing/linux/) 

```bash
sudo wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat/jenkins.io-2023.key
sudo yum upgrade
# Add required dependencies for the jenkins package
sudo yum install fontconfig java-21-openjdk
sudo yum install jenkins
```

##  2. 启动 Jenkins

```
sudo systemctl enable jenkins
sudo systemctl start jenkins
# 检查状态
sudo systemctl status jenkins
```

正常输出应显示 active (running)。

### 开放防火墙（如需）
``` bash
# 开放 8080 端口（Jenkins 默认端口）
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload
```

## 3. 配置Jenkins

### 3.1 访问 Jenkins 并完成初始化

1. 在浏览器访问：http://<服务器IP>:8080
2. 获取初始管理员密码：
   ``` bash
   sudo cat /var/lib/jenkins/secrets/initialAdminPassword
   ```
   复制输出的密码粘贴到网页。
3. 安装推荐插件：选择默认插件集（包括Git、Pipeline等常用工具）。

### 3.2 基础配置（中文支持+优化）
1. 切换为中文界面
   1. 进入 Manage Jenkins > Plugins>Available plugins，搜索 Locale 安装 "Localization: Chinese (Simplified)" 插件。
   2. 进入Manage Jenkins > Appearance， 默认语言设置为 zh_CN，勾选 "Ignore browser preference"。
2. 配置全局工具（如Git、Maven）
   1. 进入 Manage Jenkins > Global Tool Configuration：
      * Git：指定路径（默认 /usr/bin/git，或通过 which git 检查）。
      * Maven：自动安装或指定现有路径。
3. 创建管理员账户：设置用户名、密码和邮箱（避免继续使用初始密码）。
   1. * Manage Jenkins（管理 Jenkins） > Security > User > Create User
   2.  Manage Jenkins > Security 在 Authorization（授权策略） 部分选择 Matrix-based security（矩阵权限） 或 Role-Based Strategy（基于角色的权限） 需要安装授权策略插件



### 3.3 创建第一个任务（示例：执行Shell命令）
1. 点击 新建任务 > 输入任务名（如 test-job）> 选择 Freestyle project。
2. 在 构建 部分添加构建步骤：
   ``` bash
   echo "Hello, Jenkins!"
   date
   ```
3. 点击 保存 > 立即构建，在 控制台输出 中查看结果。