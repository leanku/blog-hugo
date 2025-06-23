---
title: "cloudflared 使用"
date: 2025-05-30
draft: true
categories: ["DevOps"]
tags: ["cloudflared"]
keywords: ["cloudflared"]
---

# Cloudflare/cloudflared
Cloudflare 是一家提供网络安全与性能加速服务的公司，提供了很多面向网站、API 和应用的服务。

cloudflared 是 Cloudflare 提供的开源命令行工具，主要作用是让你将本地服务、安全地暴露到公网，无需公网 IP 或端口映射。

最常见的用途：

1. Cloudflare Tunnel（以前叫 Argo Tunnel）
   * 将本地服务通过加密隧道暴露在公网。
   * 示例：把本地 http://localhost:3000 通过 Cloudflare Tunnel 映射到 https://yourdomain.com。
   * 即使服务器在内网或 NAT 后面，也能公网访问。
2. 支持 Zero Trust 应用接入
   * 配合 Cloudflare Access 实现身份验证和访问控制。

3. DNS over HTTPS (DoH) 代理
   * cloudflared proxy-dns：将本地 DNS 查询通过 Cloudflare 加密转发，提升安全性与隐私。


## 使用示例（8000端口绑定到my.example.com）
**前提条件**
* 你的域名（如 example.com）已接入 Cloudflare 并启用代理（橙色小云朵 ☁️）。
* 你已安装并配置好本地服务（如运行在 http://localhost:8000 的 Web 服务）。
* 你打算通过子域名（如 api.example.com）访问这个服务。

**一键部署脚本（CentOS）**

创建一个脚本 install_cloudflared.sh
``` shell
#!/bin/bash

set -e

# 1. 安装 cloudflared
echo "🛠️ 正在安装 cloudflared..."

# 添加 Cloudflare 官方源并安装（针对 x86_64）
sudo curl -L https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64 -o /usr/local/bin/cloudflared
sudo chmod +x /usr/local/bin/cloudflared

# 验证安装
cloudflared --version

# 2. 登录 Cloudflare
echo "🌐 请在浏览器中授权 cloudflared..."
cloudflared tunnel login

# 3. 创建 tunnel（你可以修改 tunnel 名称）
TUNNEL_NAME="my-api-tunnel"
cloudflared tunnel create $TUNNEL_NAME

# 4. 设置子域名路由（api.example.com 请换成你的）
echo "🔗 绑定子域名 my.example.com 到 tunnel..."
cloudflared tunnel route dns $TUNNEL_NAME my.example.com

# 5. 创建配置文件
CONFIG_DIR="/root/.cloudflared"
CONFIG_FILE="$CONFIG_DIR/config.yml"
CRED_FILE=$(ls $CONFIG_DIR/*.json)

echo "📝 生成配置文件：$CONFIG_FILE"

cat > $CONFIG_FILE <<EOF
tunnel: $TUNNEL_NAME
credentials-file: $CRED_FILE

ingress:
  - hostname: my.example.com
    service: http://localhost:8000
  - service: http_status:404
EOF

# 6. 创建 systemd 服务并启动
echo "🔧 设置 systemd 服务..."
cloudflared service install

echo "✅ 启动 cloudflared 服务..."
systemctl enable cloudflared
systemctl restart cloudflared

echo "🎉 Cloudflare Tunnel 部署完成，访问 https://my.example.com 即可！"
```
1. 将上面的脚本保存为 install_cloudflared.sh
2. 给脚本执行权限：```bash chmod +x install_cloudflared.sh ```
3. 执行脚本 ```bash sudo ./install_cloudflared.sh```
4. 它会提示你用浏览器打开授权链接，登录 Cloudflare 后选择你的域名授权。