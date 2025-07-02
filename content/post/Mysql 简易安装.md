---
title: "Mysql 简易安装"
date: 2024-05-11T13:46:01+08:00
draft: false
categories: ["Mysql"]
tags: ["Mysql"]
keywords: ["Mysql"]
---

# Mysql 简易安装


## 安装Mysql8.0(centos8为例)， 可参考 [MySQL官方文档](https://dev.mysql.com/doc/refman/8.0/en/linux-installation-yum-repo.html)
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

## MySQL优化
### MySQL配置优化
编辑/etc/my.cnf或/etc/mysql/my.cnf，添加/修改以下参数：
```ini
[mysqld]
# 基础配置 内存配置
# InnoDB缓冲池大小（推荐为总内存的50-70%）
innodb_buffer_pool_size = 1G
# InnoDB日志文件大小（推荐为缓冲池的25%）
innodb_log_file_size = 256M
# InnoDB日志缓冲区大小
innodb_log_buffer_size = 16M
# 查询排序缓冲区大小
sort_buffer_size = 2M
read_buffer_size = 2M
read_rnd_buffer_size = 2M
join_buffer_size = 4M

# InnoDB引擎优化
# I/O配置
innodb_flush_method = O_DIRECT
innodb_io_capacity = 1000
innodb_io_capacity_max = 2000
# 并发控制
innodb_thread_concurrency = 0
innodb_read_io_threads = 8
innodb_write_io_threads = 8
# 事务日志
innodb_flush_log_at_trx_commit = 1  # 需要最高耐久性时设为1，性能要求高时可设为2
sync_binlog = 1

# 连接相关
# 最大连接数（根据应用需求调整）
max_connections = 50
# 线程缓存大小
thread_cache_size = 10
# 表缓存
table_open_cache = 2000
table_definition_cache = 1000

# 查询优化
# query_cache_type = 0  # MySQL 8已移除查询缓存
tmp_table_size = 32M
max_heap_table_size = 32M
```

### 系统级优化
```bash
# 调整文件描述符限制
echo "* soft nofile 65535" >> /etc/security/limits.conf
echo "* hard nofile 65535" >> /etc/security/limits.conf

# 内核参数优化
echo "vm.swappiness = 10" >> /etc/sysctl.conf
echo "vm.dirty_ratio = 60" >> /etc/sysctl.conf
echo "vm.dirty_background_ratio = 5" >> /etc/sysctl.conf
echo "net.ipv4.tcp_max_syn_backlog = 65535" >> /etc/sysctl.conf
sysctl -p
```

重启，如出现错误检查日志
sudo cat /var/log/mysql/error.log