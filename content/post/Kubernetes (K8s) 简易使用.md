---
title: "Kubernetes (K8s) 简易使用 "
date: 2025-05-10T13:46:01+08:00
draft: true
categories: ["HKubernetes (K8s) 简易使用"]
tags: ["Harbor"]
keywords: ["Harbor"]
---

# Kubernetes (K8s) 简易使用

## 1. [Kubernetes (K8s)](https://kubernetes.p2hp.com/)  简介
此文适合新手快速上手，只涵盖安装、配置、基本使用

## 2. 环境
   **系统要求**
   * 操作系统：Linux（Ubuntu/CentOS）或 macOS（开发环境）
   * 内存：至少 2GB（推荐 4GB+）
   * CPU：2 核+
   * 存储：20GB+ 可用空间
  
   **安装工具**
   * Docker（K8s 依赖容器运行时）
   * kubectl（K8s 命令行工具）
   * Minikube（本地单节点 K8s，适合学习）

## 3. 前提准备
1. 关闭防火墙和SELinux
   ``` bash
   sudo systemctl stop firewalld
   sudo systemctl disable firewalld

   sudo setenforce 0
   sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

   ```
2. 关闭Swap（必须）
   ``` bash
   sudo swapoff -a
   sudo sed -i '/ swap / s/^/#/' /etc/fstab

   ```
3. 设置内核参数
``` bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

sudo sysctl --system
```

4. 安装**container runtime**（推荐使用 containerd）

   Kubernetes v1.24 开始，完全移除了 dockershim
   * 不再直接支持 Docker 作为容器运行时
   * 推荐使用 containerd 或 CRI-O
   ``` bash
   # v1.6.x（兼容 Kubernetes v1.29）
   sudo yum downgrade containerd.io-1.6.28* -y

   sudo systemctl restart containerd
   sudo systemctl enable containerd
   ls /var/run/containerd/containerd.sock  # 确认 socket 文件存在

   containerd --version  # 应显示 1.6.x
   ```

## 3. 安装 kubeadm, kubelet 和 kubectl

   ### **方式 1：Minikube（推荐新手）**
   ``` bash
   # 安装 Minikube（Linux/macOS）
   curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64  # Linux
   sudo install minikube-linux-amd64 /usr/local/bin/minikube

   # 启动 Minikube（默认使用 Docker 驱动）
   minikube start

   # 验证
   kubectl get nodes  # 应返回一个节点（STATUS=Ready）
   ```

   ### **方式 2：kubeadm（多节点集群）**
   1. 安装 kubeadm kubelet kubectl
   ```bash
   # CentOS 添加 Kubernetes 的 yum 仓库,再
      cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
   [kubernetes]
   name=Kubernetes
   baseurl=https://pkgs.k8s.io/core:/stable:/v1.29/rpm/
   enabled=1
   gpgcheck=1
   gpgkey=https://pkgs.k8s.io/core:/stable:/v1.29/rpm/repodata/repomd.xml.key
   EOF

   # 安装 kubeadm kubelet kubectl
   yum install -y kubeadm kubelet kubectl --disableexcludes=kubernetes 

   # 启动并设置开机自启 kubelet
   sudo systemctl enable --now kubelet
   # 启动并设置开机自启 kubeletcontainerd
   systemctl enable --now containerd
   ```

   2. 准备镜像，镜像拉取问题见下文中的[常见问题4和5](#image-question)
   
   3. **初始化主节点**
   
      非常关键的一步，失败会出现很多问题，可参考下文中的常见问题
      ```bash
      # **初始化主节点 此步需要确保containerd，kubelet正常运行，以及镜像已经提前拉好**
      # 参数--apiserver-advertise-address=192.168.0.2 是你的 master 节点的 IP；
      # 参数---pod-network-cidr=10.244.0.0/16 是用来配合 Flannel 网络插件的（或你改了 Calico 也行）。
      # 参数---image-repository # 使用阿里云镜像,不设置此项，就算已经拉取镜像也会因下载镜像卡主
      # 初始化成功会出现类似 Your Kubernetes control-plane has initialized successfully!
      sudo kubeadm init \
         --apiserver-advertise-address=192.168.0.2 \
         --pod-network-cidr=10.244.0.0/16 \
         --kubernetes-version=v1.29.15
      ```
   4. 配置 kubectl
      ``` bash
      mkdir -p $HOME/.kube
      sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
      sudo chown $(id -u):$(id -g) $HOME/.kube/config
      ```
   5. 安装网络插件（如 Calico）
      ``` bash
      # 安装网络插件（如 Calico）
      kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
      # 如果kubectl apply 无法应用 Calico 的 YAML 文件，可以下载文件再手动应用
         # 使用 curl
         curl -O https://mirror.aliyuncs.com/kubernetes/network-plugin/calico.yaml
         # 如果 Calico 的默认配置（如 CIDR）与你的集群不匹配，可以编辑 calico.yaml
         # 检查 CALICO_IPV4POOL_CIDR 是否与 kubeadm init --pod-network-cidr 一致（默认 192.168.0.0/16 或 10.244.0.0/16）
         kubectl apply -f calico.yaml
      ```

## 验证安装
``` bash
# 查看集群状态
kubectl cluster-info

# 检查节点状态
kubectl get nodes  # 应显示 Ready

# 检查容器运行时
kubectl get nodes -o wide | grep -i container  # 应显示 containerd://1.6.x
```

## 常用命令速查
|操作	|命令|
|:--|:--|
|查看 Pod|kubectl get pods|
|查看 Service|	kubectl get svc|
|查看日志	|kubectl logs <pod-name>|
|进入容器	|kubectl exec -it <pod-name> -- /bin/bash|
|删除 Pod	|kubectl delete pod <pod-name>|
|扩缩容	|kubectl scale --replicas=3 deployment/nginx|

## 卸载
``` bash
# Minikube
minikube stop
minikube delete

# kubeadm
sudo kubeadm reset
sudo apt-get purge kubeadm kubectl kubelet  # Ubuntu
sudo yum remove kubeadm kubectl kubelet    # CentOS
```

## 常见问题
  1. 错误显示 container runtime is not running，说明 containerd（或 Docker）未正确配置或未运行。

      **解决方法：**
      * 如果使用 containerd：
         ```bash
         # 确保 containerd 已安装并运行
         sudo systemctl enable --now containerd
         sudo systemctl status containerd

         # 生成默认配置并启用 CRI 插件
         sudo containerd config default | sudo tee /etc/containerd/config.toml
         sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
         sudo systemctl restart containerd
         ```
         * 如果使用 Docker：
         ``` bash
         # 确保 Docker 已安装并运行
         sudo systemctl enable --now docker
         sudo systemctl status docker

         # 配置 Docker 使用 systemd cgroup
         sudo mkdir -p /etc/docker
         cat <<EOF | sudo tee /etc/docker/daemon.json
         {
         "exec-opts": ["native.cgroupdriver=systemd"]
         }
         EOF
         sudo systemctl restart docker
         ```
2. 错误 /proc/sys/net/bridge/bridge-nf-call-iptables does not exist 表示系统未加载 br_netfilter 内核模块。

   **解决方法：**
   ``` bash
   # 加载内核模块
   sudo modprobe br_netfilter

   # 确保重启后依然生效
   echo "br_netfilter" | sudo tee /etc/modules-load.d/k8s.conf

   # 启用桥接流量转发
   cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
   net.bridge.bridge-nf-call-ip6tables = 1
   net.bridge.bridge-nf-call-iptables = 1
   net.ipv4.ip_forward = 1
   EOF
   sudo sysctl --system
   ```

3. kubelet 服务未能正常启动或健康检查失败

   **解决方法**
      1. 检查 kubelet 状态和日志
         ```bash
         # 查看 kubelet 运行状态
         sudo systemctl status kubelet

         # 查看详细日志（若无状态异常）
         sudo journalctl -xeu kubelet --no-pager | tail -n 50
         ```
      2. 容器运行时（CRI）未就绪
         ``` bash
         # 检查 containerd 或 Docker 是否运行
         sudo systemctl status containerd  # 或 docker

         # 如果没有运行，启动并启用
         sudo systemctl enable --now containerd
         sudo systemctl restart kubelet
         ```
4. <span id="image-question">kubeadm config images pull 失败</span>
   
   * 配置 containerd 镜像加速器
      ```bash
      # 修改 containerd 配置
      sudo mkdir -p /etc/containerd
      containerd config default | sudo tee /etc/containerd/config.toml

      # 添加国内镜像加速器（阿里云）
      sudo sed -i 's|registry.k8s.io|registry.aliyuncs.com/google_containers|g' /etc/containerd/config.toml
      sudo sed -i 's|docker.io|mirror.ccs.tencentyun.com|g' /etc/containerd/config.toml

      # 重启 containerd
      sudo systemctl restart containerd
      ```
   * 更换 Docker 仓库为国内源
      ``` bash
      # 1. 备份原有 Docker 仓库配置
      sudo mv /etc/yum.repos.d/docker-ce.repo /etc/yum.repos.d/docker-ce.repo.bak

      # 2. 使用阿里云 Docker 镜像源
      sudo tee /etc/yum.repos.d/docker-ce.repo <<'EOF'
      [docker-ce-stable]
      name=Docker CE Stable - AliYun
      baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/$releasever/$basearch/stable
      enabled=1
      gpgcheck=1
      gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg
      EOF

      # 3. 清理缓存并重新安装
      sudo yum clean all
      sudo yum makecache
      sudo yum install -y containerd.io-1.6.28
      ```
5. **当 kubeadm init 卡在 [preflight] Pulling images... 时，通常是由于 Docker/Containerd 镜像拉取失败 或 网络问题 导致**
   * 查看需要的镜像列表
      ``` bash
      kubeadm config images list --kubernetes-version=v1.29.15

      # 输出示例
      registry.k8s.io/kube-apiserver:v1.29.15
      registry.k8s.io/kube-controller-manager:v1.29.15
      registry.k8s.io/kube-scheduler:v1.29.15
      registry.k8s.io/kube-proxy:v1.29.15
      registry.k8s.io/coredns/coredns:v1.11.1
      registry.k8s.io/pause:3.9
      registry.k8s.io/etcd:3.5.16-0

      ```
   *  手动拉取镜像
      ``` bash
      sudo kubeadm config images pull --kubernetes-version=v1.29.15
      ```
      * 如果拉取失败（如网络问题），尝试：
        * 使用国内镜像源（如阿里云）
            ``` bash
            sudo kubeadm config images pull --image-repository=registry.aliyuncs.com/google_containers --kubernetes-version=v1.29.15
            ```
        * 手动拉取并重命名：
            ``` bash
            docker pull registry.aliyuncs.com/google_containers/kube-apiserver:v1.29.15
            docker tag registry.aliyuncs.com/google_containers/kube-apiserver:v1.29.15 registry.k8s.io/kube-apiserver:v1.29.15
            ```
   * 确认容器运行时
      ``` bash
      # 确认 kubelet 使用的容器运行时类型。 --container-runtime-endpoint=unix:///run/containerd/containerd.sock → 用的是 containerd
      ps -ef | grep kubelet | grep container-runtime

      # 如果是 containerd
      # 导出docker镜像
      docker save registry.k8s.io/kube-apiserver:v1.29.15 -o kube-apiserver.tar

      # 导入到 containerd：
      ctr -n k8s.io images import kube-apiserver.tar

      # 验证 containerd 中镜像是否存在：
      ctr -n k8s.io images ls | grep kube-apiserver

      # 重启 kubele
      systemctl restart kubelet
      ```
      如果想要清理 aliyun 那些 sha256 镜像
      ``` 
      ctr -n k8s.io images ls

      ctr -n k8s.io images ls | grep aliyuncs

      ctr -n k8s.io images rm <完整REF>
      ```

6. API Server 还没起来或者挂了，错误信息如： error: error validating "calico.yaml": error validating data: failed to download openapi: Get "https://192.168.0.2:6443/openapi/v2?timeout=32s": dial tcp 192.168.0.2:6443: connect: connection refused
   * 检查 kubelet 状态
      ``` bash
      systemctl status kubelet
      # 有没有绿色的 active (running)？如果不是，重启：
      systemctl restart kubelet
      ```
   * kube-apiserver 是否正常运行, 能不能看到 kube-apiserver 正在运行？如果没有，可能初始化没完成，或者配置挂了。
      ``` bash
      docker ps | grep kube-apiserver
      # 或 containerd 用户：
      crictl ps | grep kube-apiserver
      ```
   * 检查 kubectl 的配置是否 OK ``` ~/.kube/config ``` 如果没配置好,去设置
      ``` bash
      mkdir -p $HOME/.kube
      cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
      chown $(id -u):$(id -g) $HOME/.kube/config

      ```
7. **crictl** 输出报错如：ERRO[0000] validate service connection: validate CRI v1 runtime API for endpoint "unix:///var/run/dockershim.sock": rpc error: code = Unavailable desc = connection error: desc = "transport: Error while dialing: dial unix /var/run/dockershim.sock: connect: no such file or directory"
   * 说明当前 没有配置容器运行时的 socket 路径，也就是说：crictl 没法找到 containerd 或 docker 的接口来查看容器运行情况。
   * 显式配置 crictl 使用 containerd，创建/编辑配置文件
      ``` bash
      cat <<EOF | sudo tee /etc/crictl.yaml
      runtime-endpoint: unix:///run/containerd/containerd.sock
      image-endpoint: unix:///run/containerd/containerd.sock
      timeout: 10
      debug: false
      EOF
      ```
      ``` bash
      # 测试是否成功识别 containerd, 如果一切正常，会输出 containerd 的详细信息（运行中的容器、版本等）。
      crictl info
      ```
   * 检查 kubelet 是否和 containerd 联通: 编辑 /var/lib/kubelet/kubeadm-flags.env 看看是否有设置
      ``` bash
      KUBELET_KUBEADM_ARGS="--container-runtime=remote --container-runtime-endpoint=unix:///run/containerd/containerd.sock ..."

      ```
   * 然后重启 kubelet：
   ``` bash
   systemctl daemon-reexec
   systemctl restart kubelet

   ```
   * 验证测试
      ``` bash
      crictl ps | grep apiserver

      kubectl get nodes

      ```
8. Calico Pod 状态问题，如果 Pod 处于Pending、Error或者CrashLoopBackOff状态，就需要进一步排查。
要保证 Calico 相关的 Pod 都处于Running状态
   ```bash
   #查看 Calico Pod 状态
   kubectl get pods -n kube-system | grep calico

   #输出示例
   calico-kube-controllers-565f855c6c-6v9z7   1/1     Running   0          3d
   calico-node-8h5g7                          1/1     Running   0          3d
   calico-node-lt7f4                          1/1     Running   0          3d

   #  查看特定 Pod 的详细信息
   kubectl describe pod <pod-name> -n kube-system
```
* 镜像拉取失败无法拉取 Calico 镜像，可以尝试手动拉取
``` bash
# 可以手动代理下载，然后传输到目标主机导入镜像
docker pull docker.io/calico/cni:v3.25.0
docker save -o /path/to/calico-cni-v3.25.0.tar docker.io/calico/cni:v3.25.0
docker load -i /path/to/calico-cni-v3.25.0.tar

# 导入到 containerd：
ctr -n docker.io images import calico-cni-v3.25.0.tar
ctr -n docker.io images ls | grep cni
# 重启kubelet
sudo systemctl restart kubelet
```

9.  其他必要步骤
   * 注意版本兼容问题
   
   * **彻底重置并重建集群**：
      ``` bash
      # 删除文件
      sudo kubeadm reset -f
      sudo rm -rf /etc/kubernetes/ /var/lib/etcd/ ~/.kube/
      sudo systemctl restart kubelet

      # 重置
      sudo kubeadm init \
      --apiserver-advertise-address=192.168.0.2 \
      --pod-network-cidr=10.244.0.0/16 \
      --kubernetes-version=v1.29.15 \
      --image-repository=registry.aliyuncs.com/google_containers \  # 使用阿里云镜像
      --ignore-preflight-errors=Swap
      ```

   * 重置后再进行上面的如果成功初始化，记得保存 kubeadm join 命令并安装 CNI 插件
   * 重启 kubelet ``` sudo netstat -tulnp | grep 6443```
   * 确认 API Server 是否运行
      ``` bash
      # 检查 API Server 容器/Pod（根据容器运行时选择）
      sudo docker ps | grep kube-apiserver    # Docker
      sudo crictl ps | grep kube-apiserver   # Containerd
      ```
   * 检查 API Server 端口监听
      ``` bash
      sudo netstat -tulnp | grep 6443
      # 正常输出示例
      tcp  0  0  192.168.0.2:6443  0.0.0.0:*  LISTEN  <pid>/kube-apiserver
      ```
      ### 验证修复
      * 检查 kubelet 状态和日志
      * 检查容器运行时状态：
      ``` bash
      sudo crictl ps  # 或 sudo docker ps
      ```
* 检查内核模块：
   ``` bash
   lsmod | grep br_netfilter
   cat /proc/sys/net/bridge/bridge-nf-call-iptables
   ```


## 资源
* 官方文档：[kubernetes.io](kubernetes.io)
* 中文教程：[Kubernetes 中文社区](https://www.kubernetes.org.cn/)