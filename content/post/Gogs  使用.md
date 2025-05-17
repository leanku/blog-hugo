---
title: "Gogs 使用指南"
date: 2025-05-11T13:46:01+08:00
draft: false
categories: ["Gogs"]
tags: ["Gogs"]
keywords: ["Gogs"]
---

# Gogs 使用

## 1. 什么是 Gogs?
    Gogs 是一款极易搭建的自助 Git 服务。

    Gogs 的目标是打造一个最简单、最快速和最轻松的方式搭建自助 Git 服务。使用 Go 语言开发使得 Gogs 能够通过独立的二进制分发，并且支持 Go 语言支持的 所有平台，包括 Linux、Mac OS X、Windows 以及 ARM 平台。

## 2. [安装](https://gogs.io/docs/installation) Linux-amd64系统为例
### 2.1 要求
* 数据库（选择以下一项）
  * [MySQL](http://dev.mysql.com/)：版本 >= 5.7
  * [PostgreSQL](http://www.postgresql.org/)
  * [TiDB](https://github.com/pingcap/tidb)
  * 或者 什么都不安装 直接使用 SQLite3

### 2.2 下载安装
  ``` bash
  # 下载二进制
  wget https://github.com/gogs/gogs/releases/download/v0.13.2/gogs_0.13.2_linux_amd64.tar.gz

  tar -xzvf gogs_0.13.2_linux_amd64.tar.gz gogs

  cd gogs

  ./gogs web
  
  ```
### 2.3 使用 systemd服务单元文件来管理
1. 创建编辑一个服务单元文件
    ``` bash
    vim /etc/systemd/system/gogs.service
    ```
2. 在文件中添加以下内容：
    ``` bash
    [Unit]
    Description=Gogs Server
    After=network.target

    [Service]
    ExecStart=/path/gogs web
    Restart=always
    User=your_username
    Group=your_groupname

    [Install]
    WantedBy=multi-user.target
    ```
    把 /path 替换成实际的文件路径，把 your_username 和 your_groupname 替换成实际的用户名和用户组名。

3. 执行systemctl命令
    ``` bash
    # 重新加载 systemd 管理器配置
    sudo systemctl daemon-reload

    # 启动 bark-server 服务：
    sudo systemctl start bark-server

    # 设置服务开机自启：
    sudo systemctl enable bark-server

    # 查看服务状态：
    sudo systemctl status bark-server

    # 查看服务日志：
    sudo journalctl -u bark-server -f
    ```

## 3. http://localhost:3000/install 初始化配置
Gogs 默认会在端口 3000 启动 HTTP 服务，访问 /install 以进行初始配置（例如 http://localhost:3000/install ）。

## 4. 其他

### 以Mysql为例简单安装Mysql8.0， 可参考 [MySQL官方文档](https://dev.mysql.com/doc/refman/8.0/en/linux-installation-yum-repo.html)
``` bash
# CentOS el8下载 MySQL YUM 仓库（替换为最新版本）
wget https://dev.mysql.com/get/mysql80-community-release-el8-6.noarch.rpm  
# el9 使用sudo yum install mysql84-community-release-el9-x86_64.noarch.rpm
sudo rpm -Uvh mysql80-community-release-el*.rpm
sudo yum makecache
# 安装 MySQL 社区版服务器
# el9 的话sudo yum install mysql84-community-release-el9-x86_64.noarch.rpm

sudo yum install -y mysql-community-server
# 可以通过手动编辑/etc/yum.repos.d/mysql-community.repo文件来选择发布系列。
# **install时如果出现GPG 密钥问题**
# sudo curl -o /etc/pki/rpm-gpg/RPM-GPG-KEY-mysql-2023 https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
# sudo chmod 644 /etc/pki/rpm-gpg/RPM-GPG-KEY-mysql-2023
# sudo rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-mysql-2023
# 重新执行 yum install -y mysql-community-server

# 启动 MySQL 服务
sudo systemctl start mysqld
sudo systemctl enable mysqld
sudo systemctl status mysqld

#MySQL 首次启动会生成一个临时 root 密码：
sudo grep 'temporary password' /var/log/mysqld.log

# 运行安全配置向导：
#按提示操作：1.输入临时密码。2.置新密码（需符合复杂度要求）。3.移除匿名用户、禁止远程 root 登录、删除测试数据库等。
sudo mysql_secure_installation
mysql -u root -p
```
简单配置
```sql
-- 修改 root密码
ALTER USER 'root'@'localhost' IDENTIFIED BY 'MyNewPass4!';

-- 创建新用户
CREATE USER '用户名'@'%' IDENTIFIED BY '密码';

-- 设置远程访问
GRANT ALL PRIVILEGES ON *.* TO '用户名'@'%';

FLUSH PRIVILEGES;
```
防火墙3306
``` bash
sudo firewall-cmd --add-port=3306/tcp --permanent
sudo firewall-cmd --reload
```
MySQL 配置文件通常位于 /etc/my.cnf 或 /etc/mysql/my.cnf。