---
title: "使用 MinIO 自建云存储"
date: 2025-05-19T20:40:01+08:00
draft: false
categories: ["DevOps"]
tags: ["MinIO"]
keywords: ["MinIO"]
---

# 使用 MinIO 自建云存储


## 1.  MinIO 是什么
* ***定位***：一款高性能、开源的 对象存储（Object Storage）系统，完全兼容 AWS S3 API。
* ***核心目标***：提供轻量级、易部署的私有云存储方案，适用于云原生和大数据场景。
* ***开源协议***：GNU AGPL v3（商业版提供企业级支持）。
1. 核心特性
   1. 高性能:
      1. 速度优势：支持并行多线程上传/下载，单节点吞吐量可达 10-100 Gbps
      2. 低延迟: 采用 Golang 编写，优化内存管理，响应时间在毫秒级
   2. S3 完全兼容
      1. 无缝迁移：所有 AWS S3 SDK、CLI 工具（如 awscli）可直接对接 MinIO。
      2. API 支持：覆盖 Put/Get/Object、分片上传、生命周期管理等全部 S3 操作
   3. 分布式架构
      1. 纠删码（Erasure Code）：数据分片存储，允许节点故障自动恢复（如 4节点容忍2节点失效）。
      2. 多租户：支持为不同业务创建隔离的存储桶（Bucket）和访问策略。
   4. 轻量易部署
      1. 单二进制文件：无需复杂依赖，Docker/Kubernetes 一键部署。
      2. 资源占用低：单节点运行仅需 ~200MB 内存，适合边缘计算和 IoT 设备。
2. 适用场景
   1. 私有云存储
      * 替代方案：替代阿里云 OSS、AWS S3，实现数据自主可控。
      * 用例：企业文档库、备份归档。
   2. 大数据与 AI
      * 兼容性：直接对接 Hadoop HDFS、Spark、TensorFlow 的 S3 接口。
      * 用例：训练数据存储、模型版本管理。
   3. 云原生应用
      * Kubernetes 集成：通过 CSI 驱动为容器提供持久化存储。
      * 用例：微服务应用的文件共享。
   4. 边缘计算
      * 轻量级：在树莓派等设备上运行，就近处理数据。
      * 用例：物联网设备数据采集。

## 2. 部署 MinIO
1. 方式一: docker部署
    ```bash
    # 创建存储目录和配置文件
    mkdir -p ~/minio/data ~/minio/config

    # 使用 Docker 启动 MinIO（单机版）
    docker run -d \
    -p 9000:9000 \          # API 端口
    -p 9001:9001 \          # 控制台端口
    -v ~/minio/data:/data \ # 持久化存储
    -v ~/minio/config:/root/.minio \
    -e "MINIO_ROOT_USER=admin" \
    -e "MINIO_ROOT_PASSWORD=yourpassword" \
    --name minio \
    minio/minio server /data --console-address ":9001"
    ```
2. 方式二: 原生二进制安装（推荐）
   1. 下载和安装
        ```bash
        # Linux/macOS
        wget https://dl.min.io/server/minio/release/linux-amd64/minio
        chmod +x minio
        sudo mv minio /usr/local/bin/

        # 创建数据存储目录
        mkdir -p ~/minio/data

        # 启动 MinIO（前台运行，默认端口 9000）
        export MINIO_ROOT_USER=admin
        export MINIO_ROOT_PASSWORD=yourpassword
        minio server ~/minio/data \
        --address ":9002" \          # 自定义 API 端口
        --console-address ":9001"    # 自定义控制台端口
        ```
    2. 配置系统服务(可选)

        创建 Systemd 服务文件 ```/etc/systemd/system/minio.service``` 内容如下
        ```ini
        [Unit]
        Description=MinIO Object Storage
        After=network.target

        [Service]
        User=root
        Group=root
        Environment="MINIO_ROOT_USER=admin"
        Environment="MINIO_ROOT_PASSWORD=yourpassword"
        ExecStart=/usr/local/bin/minio server /path/to/minio/data \
        --console-address ":9001"
        --address ":9002"

        Restart=always
        LimitNOFILE=65536

        [Install]
        WantedBy=multi-user.target
        ```
    3. 启动服务
        ```bash
        sudo systemctl daemon-reload
        sudo systemctl enable --now minio
        # 检查状态
        sudo systemctl status minio

        # 禁用开机自启:sudo systemctl disable minio
        ```

* 访问控制台：http://服务器IP:9001，使用 admin 和 yourpassword 登录。
* 创建存储桶：
  * 进入控制台 → Buckets → Create Bucket。
  * 输入名称（如 wordpress-media），权限设为 public（或按需配置私有访问）。

##  3. 配置 HTTPS（可选但推荐）
1. 生成自签名证书
    ```bash
    mkdir -p ~/minio/certs
    openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout ~/minio/certs/private.key \
    -out ~/minio/certs/public.crt \
    -subj "/C=US/ST=State/L=City/O=Organization/CN=minio.yourdomain.com"
    ```
2. 重启 MinIO 启用 HTTPS
```bash
docker run -d \
  -p 9000:9000 \
  -p 9001:9001 \
  -v ~/minio/data:/data \
  -v ~/minio/certs:/root/.minio/certs \  # 挂载证书
  -e "MINIO_ROOT_USER=admin" \
  -e "MINIO_ROOT_PASSWORD=yourpassword" \
  --name minio \
  minio/minio server /data --console-address ":9001"
```
* 访问 HTTPS 控制台：https://服务器IP:9001（浏览器需信任自签名证书）。

## 4.WordPress 集成 MinIO
1. 安装插件
   1. 在 WordPress 后台 → 插件 → 安装插件，搜索 "WP Offload Media Lite" 并安装激活。
2. 配置插件
   1. 进入设置：
      1. WordPress 后台 → 设置 → Offload Media。
   2. 填写 MinIO 信息：
      1. Storage Provider: 选择 "Amazon S3"（MinIO 兼容 S3 API）。
      2. Access Key ID: admin（MinIO 的 root 用户）。
      3. Secret Access Key: yourpassword。
      4. Bucket: wordpress-media（提前创建的存储桶名）。
      5. Endpoint: http://服务器IP:9000（HTTPS 则填 https://域名:9000）。
      6. Region: 留空或填 us-east-1（MinIO 忽略此字段）。
   3. 高级设置：
      1. 勾选 "Enable Path Style Access"（MinIO 必须开启）。
      2. 设置 "File URL" 为 http://服务器IP:9000/wordpress-media（或自定义域名）。
3. 测试上传
   1. 在 WordPress 媒体库上传图片，检查是否自动同步到 MinIO 存储桶：
    ```bash
    ls ~/minio/data/wordpress-media  # 查看文件是否生成
    ```
## 5. 优化
1. 绑定域名并启用 CDN
   1. 域名解析：将 media.yourdomain.com 解析到服务器 IP。
   2. Nginx 反向代理（示例配置）：
        ```nginx
        server {
            listen 80;
            server_name media.yourdomain.com;
            return 301 https://$host$request_uri;
        }

        server {
            listen 443 ssl;
            server_name media.yourdomain.com;

            ssl_certificate /etc/letsencrypt/live/media.yourdomain.com/fullchain.pem;
            ssl_certificate_key /etc/letsencrypt/live/media.yourdomain.com/privkey.pem;

            location / {
                proxy_pass http://localhost:9000;
                proxy_set_header Host $host;
            }
        }
        ```
    3. 更新 WordPress 文件 URL：
       1. 在 WP Offload Media 设置中修改为 https://media.yourdomain.com/wordpress-media。
2. 权限控制
   1. 私有文件访问：
      1. 在 MinIO 控制台将存储桶设为 private。
      2. 使用插件生成预签名 URL（需 Pro 版功能）。
3. 监控与备份
   1. 监控：通过 MinIO 控制台的 Metrics 查看流量和存储。
   2. 备份：定期压缩 ~/minio/data 目录或使用 mc mirror 命令同步到其他存储。
4. 通过 CDN 加速全球访问（如 Cloudflare）。

## 6. MinIO 的通用兼容性

MinIO 完全兼容 AWS S3 API，因此任何支持 S3 协议的工具、语言或框架均可直接调用。以下是集成方式(php)

   1. 使用 MinIO 官方 PHP SDK [https://min.io/docs/minio/linux/developers/minio-drivers.html?ref=docs](https://min.io/docs/minio/linux/developers/minio-drivers.html?ref=docs)
   
        ```bash
        composer require minio/minio
        ```
        ```php
        <?php
        require 'vendor/autoload.php';

        use Minio\MinioClient;

        // 初始化客户端
        $minio = new MinioClient([
            'endpoint' => 'minio服务器IP:9000',
            'accessKey' => 'admin',
            'secretKey' => 'yourpassword',
            'useSSL' => false // 如果是 HTTPS 则改为 true
        ]);

        // 上传文件
        try {
            $minio->putObject([
                'Bucket' => 'wordpress-media',
                'Key'    => 'uploads/test.jpg',
                'SourceFile' => '/var/www/html/test.jpg',
            ]);
            echo "上传成功";
        } catch (Exception $e) {
            echo "错误: " . $e->getMessage();
        }
        ```

## 7. 其他同类工具
1. Ceph
2. SeaweedFS
3. GlusterFS