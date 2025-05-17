---
title: "K3s使用指南 +Rancher"
date: 2025-05-12T21:46:01+08:00
draft: false
categories: ["DevOps"]
tags: ["k3s"]
keywords: ["k3s"]
---

# K3s使用指南+Rancher

## 1. [K3s](https://k3s.io)  简介
* K3s 是一个轻量级的 Kubernetes 发行版，由 Rancher Labs（现在是 SUSE 旗下）开发。
* 它完全兼容 Kubernetes API，但设计上更轻便、易安装，适合边缘计算、物联网设备、单节点或资源有限环境。
* K3s 把 Kubernetes 的很多组件做了简化，比如内置了 containerd，默认启用 flannel 网络，去掉了复杂的插件，安装非常简单。
* 主要目标是让 Kubernetes 快速部署、低资源占用，并且更适合国内和小型集群使用。


## 2. K3s 对比 kubeadm
| 特性/方面       | K3s      | kubeadm   |
| ----------- | ---------| ---------------------------- |
| **定位**      | 轻量级、开箱即用的 Kubernetes 发行版    | 官方工具，用于标准 Kubernetes 集群的快速部署和引导    |
| **安装复杂度**   | 极简安装，单条命令搞定                                                  | 需要多个步骤，配置复杂，适合有一定 Kubernetes 经验的用户                                                                                                               |
| **组件集成**    | 集成了 `containerd`，默认内置 `flannel` 网络，默认关闭了部分复杂组件（如部分云插件、Helm等） | 只负责初始化集群，组件和网络插件需要用户自行选择安装                                                                                                                       |
| **资源占用**    | 非常低，适合边缘设备、物联网、单机小集群                                         | 资源占用较大，适合生产多节点环境                                                                                                                                 |
| **多节点支持**   | 支持多节点，但更适合轻量和小规模集群                                           | 原生支持多节点和大规模集群，灵活度高                                                                                                                               |
| **适用场景**    | 单机、开发测试、小型集群、资源受限环境                                          | 生产环境，多节点，企业级集群部署                                                                                                                                 |
| **网络插件**    | 默认集成 Flannel，安装简单                                            | 用户需自行部署网络插件（Flannel、Calico、Weave等）                                                                                                               |
| **更新升级**    | 版本更新简单，内置自动化升级工具                                             | 需要手动升级，过程复杂                                                                                                                                      |
| **集成工具和生态** | 内置 Traefik（可选关闭），轻量且默认功能有限                                   | 灵活，可按需安装 Ingress、Dashboard、Helm 等组件                                                                                                              |
| **社区和支持**   | Rancher 支持，社区活跃                                              | CNCF 官方支持，社区广泛                                                                                                                                   |
| **官网文档**    | [https://k3s.io](https://k3s.io)                             | [https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/) |

* 如果你是想快速搭建一个单机或轻量集群，或者资源有限，想要简单易用、快速上手，推荐用 K3s。

* 如果你想搭建一个生产级多节点集群，或者需要最大程度自定义集群组件和网络插件，且不介意配置复杂，推荐用 kubeadm。

## 3. 安装 K3s 
可以使用下面的shell
``` bash
#!/bin/bash

set -e

echo "开始安装 K3s（国内镜像加速）..."

# 1. 设置国内镜像环境变量，自动使用阿里云镜像源
export INSTALL_K3S_MIRROR=cn

# 2. 下载并安装 K3s，禁用内置 Traefik（减少资源占用）
INSTALL_K3S_MIRROR=cn curl -sfL https://get.k3s.io | sh -s - --flannel-backend=host-gw --disable traefik --write-kubeconfig-mode 644

if [ $? -eq 0 ]; then
  echo "✅ K3s 安装成功！"
else
  echo "❌ K3s 安装失败，请检查日志。"
  exit 1
fi

# 3. 设置 kubectl 配置环境变量，方便后续使用
echo "export KUBECONFIG=/etc/rancher/k3s/k3s.yaml" >> ~/.bashrc
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml

# K3s 的 kubectl 没有加入环境变量
echo 'export PATH=$PATH:/usr/local/bin' >> ~/.bashrc
source ~/.bashrc

# 4. 显示节点状态，确认集群正常运行
echo "等待集群启动完成，3秒后自动查询节点状态..."
sleep 3
systemctl status k3s

kubectl get pods -n kube-system -o wide

kubectl get nodes

echo "安装完成。你可以使用 kubectl 操作集群了。"

```
1. 复制脚本内容保存成 install-k3s.sh
2. chmod +x install-k3s.sh
3. ./install-k3s.sh
4. 安装完成后，每次登录终端都自动设置 KUBECONFIG，可直接运行 kubectl 命令

**常见问题**
-------
   1. failed to pull image "rancher/mirrored-pause:3.6"
      ``` bash
      # 编辑 k3s 服务文件 /etc/systemd/system/k3s.service
      K3S_EXEC="--pause-image=registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.6 --disable traefik --write-kubeconfig-mode 644"

      sudo systemctl daemon-reload
      sudo systemctl restart k3s
      ```
      **配置 containerd（K3s 默认使用 containerd）拉取镜像的代理地址。**
      编辑 /etc/rancher/k3s/registries.yaml 文件（如果不存在可创建）
      ```yaml
      mirrors:
      "docker.io":
      endpoint:
         - "https://registry.aliyuncs.com"

      ```
      重启 K3s：```systemctl restart k3s```

   2. K3s 默认内置 flannel 作为 CNI，如果 flannel 没启动或者失败，会导致 Pod 无法获得网络
      ```bash
      # CNI 插件的 Pod 状态
      kubectl get pods -n kube-system -o wide | grep flannel

      # 查看 K3s 服务日志
      sudo journalctl -u k3s -f
      ```

## 4. K3s 管理集群
### 1. 查看集群状态
   ``` bash
   kubectl get nodes           # 查看集群中节点状态
   kubectl get pods -A         # 查看所有命名空间的 Pod 状态
   kubectl cluster-info        # 查看集群信息
   ```
### 2. 部署应用
用标准 Kubernetes YAML 文件部署应用，例如 nginx：
   ``` yaml
   apiVersion: v1
   kind: Pod
   metadata:
   name: nginx
   spec:
   containers:
      - name: nginx
         image: nginx:latest
         ports:
         - containerPort: 80
   ```
   保存为 nginx.yaml，然后：
   ``` bash
   kubectl apply -f nginx.yaml
   kubectl get pods
   ```
### 3. 管理服务
:创建 Service 公开 Pod：
   ``` yaml
   apiVersion: v1
   kind: Service
   metadata:
   name: nginx-service
   spec:
   selector:
      app: nginx
   ports:
      - protocol: TCP
         port: 80
         targetPort: 80
   type: NodePort
   ```
### 4. 查看日志和调试
   ``` bash
   kubectl logs <pod-name>          # 查看指定 Pod 日志
   kubectl describe pod <pod-name>  # 查看 Pod 详情
   kubectl exec -it <pod-name> -- /bin/sh  # 进入 Pod 容器
   ```
### 5. **卸载 K3s**
:如果需要完全卸载 K3s，执行：
   ``` bash
   /usr/local/bin/k3s-uninstall.sh
   ```
## 5. 提示
* Kubeconfig 文件路径：/etc/rancher/k3s/k3s.yaml
* 默认网络插件：flannel，开箱即用
* 如果需要 Ingress，可以自行安装 Nginx Ingress Controller，或者启用 Traefik（默认禁用）


## 6. 安装[Rancher](https://ranchermanager.docs.rancher.com/zh/)
* Rancher 是一个 Kubernetes 管理工具，让你能在任何地方和任何提供商上部署和运行集群。(页面和文档支持中文)

* 资源占用较高：镜像~1.1GB+，启动慢一些；内存空闲时约： 400~800MB

* 资源紧张可以用Portainer代替，下文有介绍
1. 使用docker安装
   ``` bash
   docker run -d --restart=unless-stopped \
   --privileged \
   -p 8443:8443 \
   --name rancher \
   -v /develop/data/rancher:/var/lib/rancher \
   -v /etc/ssl/certs:/etc/ssl/certs \
   rancher/rancher:latest
   ```
   如果拉不到可配置docker镜像加速
   编辑 /etc/docker/daemon.json：
   ```bash
   {
   "registry-mirrors": [
      "https://registry.docker-cn.com",
      "https://docker.mirrors.ustc.edu.cn",
      "https://hub-mirror.c.163.com"
   ]
   }
   ```
   然后重启docker：
   ```bash
   sudo systemctl daemon-reexec
   sudo systemctl restart docker
   ```
2. 然后浏览器访问：https://<你的服务器IP>:8443
   1. 开放8443 端口
   2. 首次登录会提示设置管理员密码。设置完毕后，右上角点击头像 → Language → 选择“中文（简体）”。
3. Rancher 连接并管理现有的 K3s 集群
   1. 登录 Rancher 后台
   2. 添加已有集群（导入现有 K3s）
      1. 左侧点击「集群管理」 > 「创建集群」
      2. 选择右侧的 导入现有集群（Import an Existing Cluster）
      3. 输入一个你喜欢的名称，例如 k3s-01
      4. 点击「创建」，你会看到一个命令：
         ``` bash
         kubectl apply -f https://<rancher-server>/v3/import/xxxx.yaml
         ```
         复制这个命令。
      5. 在你的 K3s 集群主节点 上执行 Rancher 提供的导入命令。例如：
         ```bash
         kubectl apply -f https://<rancher-server>/v3/import/xxxx.yaml
         ```
         * 执行命令的节点要能通过公网访问 Rancher（https://<Rancher IP>:8443），否则你可以改用 IP 并加上 --insecure-skip-tls-verify。
         * 如果你用的是 root 用户安装的 K3s，Kubeconfig 默认在 /etc/rancher/k3s/k3s.yaml，你可以这样指定：
            ```bash
            KUBECONFIG=/etc/rancher/k3s/k3s.yaml kubectl apply -f ...
            ```
      6. 等待连接成功
   
         回到 Rancher Web UI，等待 30 秒左右，系统会检测到该 K3s 集群，状态变为 Active（运行中），表示集群已被成功接管！
         你可以：
            * 浏览命名空间、Pod、节点
            * 执行命令行
            * 安装应用（例如 Prometheus、Helm Chart）
            * 设置权限和监控

**提示**
* Rancher 只作为控制面板，不会修改你 K3s 的配置。
* 你可以导入多个 K3s 集群。
* 如果你在国内访问 Rancher 速度慢，可配置 nginx 反向代理 + 内网地址。

## 7. 安装[Portainer](https://docs.portainer.io)
1. 确定你的 K3s 已安装并运行
   ```bash
   k3s kubectl get nodes
   # 确认节点 Ready，就说明集群正常
   ```
2. K3s 安装完后默认把 kubeconfig 文件放在``` /etc/rancher/k3s/k3s.yaml```
   
   你可以将它复制到一个用户可访问的位置 比如：
   ```bash
   mkdir -p ~/portainer/kube
   cp /etc/rancher/k3s/k3s.yaml ~/portainer/kube/config

   ```
   然后把文件中的 server 地址从 127.0.0.1 改成主机 IP，比如：
   ```bash
   server: https://192.168.0.2:6443
   # 必须改，否则 Portainer 容器内连不到主机的 K3s API
   ```

3. 用 Docker 启动 Portainer
   ```bash
   docker run -d \
   -p 9000:9000 \
   -p 9443:9443 \
   --name portainer \
   --restart=always \
   -v /var/run/docker.sock:/var/run/docker.sock \
   -v /develop/data/portainer-data:/data \
   -v ~/portainer/kube:/kube \
   portainer/portainer-ce

   ```
   * /develop/data/portainer-data → 保存 Portainer 的数据
   * ~/portainer/kube:/kube → 挂载包含 k3s.yaml 的目录

4. 访问 Portainer UI
   * 浏览器打开：http://<你的服务器IP>:9000
   * 第一次登录会让你创建管理员账户。
   * 如果页面提示超时timed out ,重启portainer ```docker restart portainer```
