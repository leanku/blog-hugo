---
title: "基于 Gogs + Jenkins + Harbor + Docker 的自动化部署方案"
date: 2025-05-17T18:46:01+08:00
draft: false
categories: ["DevOps"]
tags: ["DevOps"]
keywords: ["DevOps","Jenins","Gogs","Harbor"]
---

# 基于 Gogs + Jenkins + Harbor + Docker 的自动化部署方案

## 1. 系统架构总览

``` mermaid
开发者提交代码 → Git仓库 → Jenkins触发构建 → Docker构建镜像 → 推送至Harbor → 
Kubernetes部署更新 → 监控反馈
```

## 2. 环境准备
### 2.1 硬件要求
* 最低配置：2核CPU/4GB内存/100GB存储
* 推荐配置：4核CPU/8GB内存/200GB SSD


## 3. 组件安装与配置
### 3.1 Gogs 安装
另外一篇[Jenkins 使用](/content/post/Jenkins%20使用.md)

### 3.2 [Jenkins](https://www.jenkins.io/doc/book/installing/linux/) 安装
另外一篇[Gogs  使用](/content/post/Gogs%20%20使用.md)

### 3.3 Harbor 安装
另外一篇[Harbor 使用](/content/post/Harbor%20使用.md)

## 4. 环境配置
下面以wordpress项目为例
```
wordpress/
├── app/                  # 项目代码
├── docker                # Docker 相关文件
|    ├── Dockerfile      
|    ├── entrypont.sh      
|    └── nginx.conf       
|    └── nginx-wordpress.conf 
├── Jenkinsfile           # Jenkins Pipeline 定义
└── k8s/                  # Kubernetes 部署文件
    ├── deployment.yaml   # Deployment 配置
    ├── service.yaml      # Service 配置
    └── ingress.yaml      # Ingress 配置（可选）
    └── namespace.yaml    # namespace 配置（可选）
```
### 4.1 JenkinsPipeline配置
1. 安装必要插件
* Kubernetes Continuous Deploy
* Docker Pipeline
* Gogs plugin
* Harbor
* Git

2. 配置凭据: 
Manage Jenkins > Credentials
* Git仓库访问凭据 ID如 gogs-credential
* Harbor仓库凭据 ID如harbor-credential
* Kubernetes集群凭据  （/etc/rancher/k3s）ID如 k3s-kubeconfig


Jenkins 示例：
```
```

### 4.2 Harbor配置
1. 创建专门的项目(如wordpress)
2. 配置访问权限
3. 确保Jenkins服务器可以访问Harbor API

如果存在证书问题

  检查 Docker 配置文件/etc/docker/daemon.json加入
  ```bash
  {
    "insecure-registries": ["IP地址"]
  }
  ```

  ```bash
  # 下载Harbor证书（需替换实际路径）
  sudo mkdir -p /etc/docker/certs.d/ip
  sudo curl -k https://ip/api/v2.0/systeminfo/getcert -o /etc/docker/certs.d/ip/ca.crt

  # 更新系统证书库
  sudo cp /etc/docker/certs.d/ip/ca.crt /usr/local/share/ca-certificates/
  sudo update-ca-certificates

  # 重启docker
  sudo systemctl restart docker
  ```

### 4.3 Gogs Webhook配置
1. 在Gogs仓库设置中管理Web钩子：
    ```
    URL如: http://ip:端口默认8080/gogs-webhook/?job=jobnNme
    Secret: your-shared-secret
    触发事件: 推送、提交
    ```

### 4.4 Jenkins Pipeline设计
``` Groovy

def getTriggerUser() {
    def causes = currentBuild.getBuildCauses()
    if (causes) {
        def userCause = causes.find { it.shortDescription?.contains("User") }
        if (userCause) {
            // 从 shortDescription 中提取用户名（格式通常为 "Started by user <用户名>"）
            def match = userCause.shortDescription =~ /Started by user (.*)/
            return match ? match[0][1] : "未知用户"
        }
    }
    return '系统触发'
}

// Jenkinsfile
pipeline {
  agent any

  // 1. 定义
  environment {
    // 应用名称
    APP_NAME = "wordpress"

    // Harbor 镜像仓库地址
    REGISTRY = "host/wordpress"

    // K3s 集群上下文名称 (根据您的 kubeconfig 配置 cat /etc/rancher/k3s/k3s.yaml)
    KUBE_CONTEXT = "default"
  }

  // 2. 拉取代码
  stages {
    stage('Checkout') {
      steps {
        // 从 Gogs 拉取代码
        git branch: 'master',
           url: 'host/username/wordpress.git',
           credentialsId: 'gogs-credential'  // 预先配置的 Gogs 访问凭证
      }
    }

    // 3. 定义镜像标签
    stage('Prepare') {
        steps {
          script {
            // 获取 Git 提交信息（确保代码已 checkout）
            env.GIT_COMMIT_SHORT = sh(
              script: 'git rev-parse --short HEAD',
              returnStdout: true
            ).trim()
            // 定义镜像标签
            env.IMAGE_TAG = "build-${env.BUILD_NUMBER}-${env.GIT_COMMIT_SHORT}"
          }
        }
      }

    // 4. 构建镜像 并 推送镜像
    stage('Build Image') {
      steps {
        script {
          // 构建 Docker 镜像 (使用 Dockerfile 中的定义)
          docker.build("${REGISTRY}/${APP_NAME}:${env.IMAGE_TAG}", "-f docker/Dockerfile .")

          // 使用环境变量安全登录并推送
          withCredentials([usernamePassword(
            credentialsId: 'harbor-credential',
            usernameVariable: 'HARBOR_USER',
            passwordVariable: 'HARBOR_PASS'
          )]) {
            sh """
              # 登录Harbor
              echo \$HARBOR_PASS | docker login ${env.REGISTRY} -u \$HARBOR_USER --password-stdin

              # 推送带构建标签的镜像
              docker push ${env.REGISTRY}/${env.APP_NAME}:${env.IMAGE_TAG}

              # 额外推送latest标签
              docker tag ${env.REGISTRY}/${env.APP_NAME}:${env.IMAGE_TAG} ${env.REGISTRY}/${env.APP_NAME}:latest
              docker push ${env.REGISTRY}/${env.APP_NAME}:latest
            """
          }
        }
      }
    }

    // 5. k3s 部署
    stage('Deploy to K3s') {
      steps {
        script {
          // 使用 kubeconfig 凭证
          withCredentials([file(
            credentialsId: 'k3s-kubeconfig',
            variable: 'KUBECONFIG'
          )]) {
            // 设置 KUBECONFIG 环境变量
            withEnv(["KUBECONFIG=${env.KUBECONFIG}"]) {
              // 应用 Kubernetes 配置
              sh """
                kubectl config use-context ${KUBE_CONTEXT} || exit 1

                # 检查集群连接
                kubectl cluster-info || exit 1

                # 创建命名空间
                kubectl apply -f k8s/namespace.yaml || exit 1

                # 部署应用（确保文件按顺序应用）
                kubectl apply -f k8s/deployment.yaml || exit 1
                kubectl apply -f k8s/service.yaml || exit 1
                # 当前 NodePort模式不用ingress
                # kubectl apply -f k8s/ingress.yaml || exit 1

                # 更新镜像 (使用带构建ID的标签)
                kubectl set image deployment/wordpress \
                  wordpress=${REGISTRY}/${APP_NAME}:${env.IMAGE_TAG} \
                  -n wordpress || exit 1

                # 等待部署完成
                kubectl rollout status deployment/wordpress -n wordpress --timeout=300s || exit 

                # 检查状态
                kubectl get pods,svc,ing -n wordpress
              """
            }
          }
        }
      }
    }
  }


  post {
    // 成功
    success {
      script {
        def successMessage = """
        • *Jenkins Build Success*
        • 项目: ${env.JOB_NAME} #${env.BUILD_NUMBER}
        • 状态: ${currentBuild.result}
        • 耗时: ${currentBuild.durationString}
        • 镜像: ${env.REGISTRY}/${env.APP_NAME}:${env.IMAGE_TAG}
        • 详情: ${env.BUILD_URL}
        """.stripIndent().trim()

        // 发送 Slack 通知（成功）需要Slack Notification 插件
         slackSend(
           channel: env.SLACK_CHANNEL,
           color: 'good',
           message: successMessage
         )
      }
    }
    // 失败
    failure {
      script {
        def triggerUser = getTriggerUser()
        def failureMessage = """
          • *Jenkins Build Failed*
          • 项目: ${env.JOB_NAME} #${env.BUILD_NUMBER}
          • 状态: ${currentBuild.result}
          • 执行人: ${triggerUser}
          • 建议: 检查日志或联系 DevOps
        """.stripIndent().trim()

      // 发送 Slack 通知（失败）
      slackSend(
         channel: env.SLACK_CHANNEL,
         color: 'danger',
         message: failureMessage
       )
      }
   }
   
    always {
      // 清理工作空间 需要Workspace Cleanup 插件
      cleanWs()
    }
  
  }
}
```

### 4.5 Kubernetes部署文件
k8s/namespace.yaml
```yaml
# k8s/namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
name: wordpress
```
k8s/service.yaml
```yaml
# k8s/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: wordpress
  namespace: wordpress
spec:
  # 选择器（匹配Pod标签）
  selector:
    app: wordpress
  # 端口配置
  ports:
    - name: http
      protocol: TCP
      port: 80          # 服务端口
      targetPort: 80    # 容器端口
      nodePort: 30080   # 手动指定端口（建议选 30000 以上未占用端口）
  type: NodePort        
```
k8s/deployment.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress
  namespace: wordpress
  labels:
    app: wordpress
spec:
  replicas: 1  # 实例数量
  selector:
    matchLabels:
      app: wordpress
  strategy:
    rollingUpdate:
      maxSurge: 1       # 升级期间允许超过期望值的 Pod 数量
      maxUnavailable: 0  # 升级期间不可用 Pod 数量
  template:
    metadata:
      labels:
        app: wordpress
    spec:
      terminationGracePeriodSeconds: 90  # 延长终止宽限期
      containers:
        - name: wordpress
          image: host/wordpress/wordpress:latest  # 镜像会被 Jenkins 动态替换
          ports:
            - containerPort: 80
          env:
            - name: WORDPRESS_DB_HOST
              value: host:port
            - name: WORDPRESS_DB_USER
              value: super
            - name: WORDPRESS_DB_PASSWORD
              valueFrom:
                secretKeyRef:                 #指定连接 MySQL 的密码（从 Kubernetes Secret 获取）需提前创建namespace：kubectl create namespace 创建Secret命令:kubectl create secret generic mysql-secret --from-literal=password='your_secure_password' -n wordpress
                  name: mysql-secret          # Secret 名称
                  key: password               # Secret 中的键名
            - name: WORDPRESS_DB_NAME
              value: "wordpress"
          resources:
            requests:
              cpu: "100m"
              memory: "128Mi"
            limits:
              cpu: "500m"
              memory: "512Mi"
          livenessProbe:
            httpGet:
              path: /wp-login.php       # 根路径（无需域名解析）
              port: 80                  # 容器内服务端口
              scheme: HTTP              # 协议与容器内服务一致（HTTP/HTTPS）
            initialDelaySeconds: 60     # 容器启动后等待 30 秒再开始探测
            periodSeconds: 20           # 每 10 秒探测一次
            timeoutSeconds: 5           # 超时时间 5 秒
            failureThreshold: 3         # 连续 3 次失败则判定为不健康
          readinessProbe:
            httpGet:
              path: /wp-login.php
              port: 80
              scheme: HTTP
            initialDelaySeconds: 30
            periodSeconds: 10
            timeoutSeconds: 5
            failureThreshold: 3
      imagePullSecrets:
        - name: harbor-secret   

```

## 5. 完整工作流程总结
* 开发者推送代码到Git仓库
* Jenkins检测到变更并触发Pipeline
* Pipeline执行以下步骤:
  * 拉取最新代码
  * 构建Docker镜像
  * 运行单元测试(可选)
  * 推送镜像到Harbor
  * 更新Kubernetes部署
  * 执行健康检查
  * 部署成功后通知
  * 监控系统跟踪应用性能