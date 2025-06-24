---
title: "SSH 使用指南"
date: 2025-05-11T13:46:01+08:00
draft: false
categories: ["SSH"]
tags: ["SSH"]
keywords: ["SSH"]
---

# SSH 使用指南：从基础到服务器密钥登录

## 什么是 SSH？

SSH (Secure Shell) 是一种加密的网络协议，用于在不安全的网络中安全地进行远程登录和其他网络服务。它通过加密技术保护通信安全，防止信息泄露和中间人攻击，是系统管理员和开发人员管理远程服务器的必备工具。

## 基本 SSH 连接

最基本的 SSH 连接命令格式如下：
```bash
ssh username@hostname
```

例如，要连接到 IP 为 192.168.1.100 的服务器，用户名为 admin：
```bash
ssh admin@192.168.1.100
```

首次连接时会提示验证服务器指纹：

```text
The authenticity of host '192.168.1.100 (192.168.1.100)' can't be established.
ECDSA key fingerprint is SHA256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.
Are you sure you want to continue connecting (yes/no)?
```

输入 `yes` 后，服务器指纹会被保存在本地 `~/.ssh/known_hosts` 文件中，以后连接时会自动验证。

## SSH 配置文件

SSH 客户端配置文件 `~/.ssh/config` 可以简化连接命令。例如：
```text
Host myserver
    HostName 192.168.1.100
    User admin
    Port 2222
```

配置后，只需输入 `ssh myserver` 即可连接。

## 服务器密钥登录

相比密码登录，使用 SSH 密钥对更安全且方便。以下是设置步骤：

### 1\. 生成密钥对

在本地机器上生成 RSA 密钥对（默认 3072 位，推荐使用 ed25519 算法更安全）：
```bash
ssh-keygen \-t ed25519 \-C "your\_email@example.com"
```

或使用 RSA：
```bash
ssh-keygen \-t rsa \-b 4096 \-C "your\_email@example.com"
```
生成过程中会提示输入保存位置（默认为 `~/.ssh/id_ed25519` 或 `~/.ssh/id_rsa`）和密码（可选）。

### 2\. 将公钥上传到服务器

方法一：使用 `ssh-copy-id` 命令（最简单）：
```bash
ssh-copy-id \-i ~/.ssh/id\_ed25519.pub username@hostname
```

方法二：手动复制（如果没有 `ssh-copy-id`）：

```bash
cat ~/.ssh/id\_ed25519.pub | ssh username@hostname "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized\_keys"
```

### 3\. 设置服务器权限

确保服务器上 `.ssh` 目录和 `authorized_keys` 文件权限正确：

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized\_keys
```

### 4\. 禁用密码登录（可选，推荐）

编辑服务器上的 `/etc/ssh/sshd_config` 文件：
```text
PasswordAuthentication no
PubkeyAuthentication yes
```

然后重启 SSH 服务：
```bash
sudo systemctl restart sshd
```


## 高级 SSH 功能

### 端口转发

本地端口转发（将远程服务器端口映射到本地）：

```bash
ssh \-L 8080:localhost:80 username@hostname
```
远程端口转发（将本地端口映射到远程服务器）：

```bash
ssh \-R 8080:localhost:80 username@hostname
```

### SCP 文件传输

从本地复制到远程：
```bash
scp file.txt username@hostname:/path/to/destination
```
从远程复制到本地：

```bash
scp username@hostname:/path/to/file.txt /local/destination
```

### SFTP 文件传输

启动 SFTP 会话：

```bash

sftp username@hostname
```

### SSH 代理转发

启用代理转发可以在连接多个服务器时不用每次都使用密钥：

```bash

ssh \-A username@hostname
```
或在 `~/.ssh/config` 中配置：

```text
Host \*
    ForwardAgent yes
```

## 安全最佳实践

1.  **使用强密码或密钥**：密钥比密码更安全，尤其是 ed25519 或 RSA 4096 位
    
2.  **禁用 root 登录**：修改 `/etc/ssh/sshd_config` 中 `PermitRootLogin no`
    
3.  **更改默认端口**：修改 `/etc/ssh/sshd_config` 中 `Port 2222`（或其他非22端口）
    
4.  **使用 fail2ban**：防止暴力破解攻击
    
5.  **定期更新密钥**：建议每6-12个月更换一次密钥
    
6.  **限制用户访问**：使用 `AllowUsers` 指令限制可登录的用户
    

## 常见问题解决

**问题1**：连接时出现 "WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!"

**解决**：服务器密钥已更改（可能是重装系统），删除 `~/.ssh/known_hosts` 中对应条目。

**问题2**：密钥登录失败

**解决**：

1.  检查 `authorized_keys` 文件权限是否为 600
    
2.  检查 `.ssh` 目录权限是否为 700
    
3.  检查服务器 `/etc/ssh/sshd_config` 中 `PubkeyAuthentication` 是否为 yes
    

**问题3**：SSH 连接超时

**解决**：

1.  检查网络连接和防火墙设置
    
2.  确认 SSH 服务正在运行 `sudo systemctl status sshd`
    
3.  检查服务器是否监听正确端口 `netstat -tuln | grep ssh`
    

通过掌握这些 SSH 使用技巧和密钥登录方法，你可以更安全、高效地管理远程服务器。密钥登录不仅提高了安全性，还免去了记忆复杂密码的麻烦，是专业运维人员的首选认证方式。

