---
title: "使用Bark 实现IOS接收通知"
date: 2025-05-11T20:46:01+08:00
draft: false
categories: ["Bark"]
tags: ["Bark"]
keywords: ["Bark"]
---

# Bark 实现IOS接收通知

## 1. Bark介绍
    免费、轻量！简单调用接口即可给自己的iPhone发送推送。
    依赖苹果APNs，及时、稳定、可靠
    不会消耗设备的电量， 基于系统推送服务与推送扩展，APP本体并不需要运行。
    隐私安全，可以通过一些方式确保包含作者本人在内的所有人都无法窃取你的隐私。

## 2. [安装](https://bark.day.app/#/tutorial) 
### 2.1 AppStore 下载Bark 允许通知权限
替换内容去请求给的地址即可接收通知，各参数可参考文档

## 2 也可自己部署服务端 bark-server

### 2.2 下载安装 [bark-server](https://github.com/Finb/bark-server/releases)
  ``` bash
  # 下载二进制，不同系统注意更换版本
  wget https://github.com/Finb/bark-server/releases/download/v2.2.0/bark-server_linux_amd64

  chmod +x bark-server_linux_amd64

  ./bark-server_linux_amd64 -addr 0.0.0.0:8080 -data ./bark-data

  ```

## 3. 使用 systemd服务单元文件来管理 bark-server 的运行
1. 创建编辑一个服务单元文件
    ``` bash
    vim /etc/systemd/system/bark-server.service
    ```
2. 在文件中添加以下内容：
    ``` bash
    [Unit]
    Description=Bark Server
    After=network.target

    [Service]
    ExecStart=/path/to/your/bark-server_linux_amd64 -addr 0.0.0.0:8080 -data /path/to/your/bark-data
    Restart=always
    User=your_username
    Group=your_groupname

    [Install]
    WantedBy=multi-user.target
    ```
    把 /path/to/your/bark-server_linux_amd64 和 /path/to/your/bark-data 替换成实际的文件路径，把 your_username 和 your_groupname 替换成实际的用户名和用户组名。

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
4. 启动成功后 打开iPhone上的Bark添加服务器地址，即可完成自部署