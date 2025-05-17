---
title: "Prometheus + Grafana + Alertmanager轻量级监控告警系统"
date: 2025-05-11
draft: false
categories: ["监控告警"]
tags: ["监控告警"]
keywords: ["监控告警"]
---

# Prometheus + Grafana + Alertmanager轻量级监控告警系统


## **一、工具简介与官网**

|工具|作用|官网文档地址|
|:--|:--|:--|
|**Prometheus** | 指标采集、存储与告警规则管理|[prometheus.io/docs](https://prometheus.io/docs/)|
|**Node Exporter**|暴露服务器硬件/OS指标|[github.com/prometheus/node\_exporter](https://github.com/prometheus/node_exporter)|
|**Alertmanager**|告警聚合与通知发送|[prometheus.io/docs/alerting](https://prometheus.io/docs/alerting/latest/alertmanager/)|
|**Grafana**|数据可视化与仪表盘|[grafana.com/docs](https://grafana.com/docs/)|




## **二、安装与配置**


### **1\. Prometheus 安装**

#### **1.1 下载与解压**
``` bash

wget https://github.com/prometheus/prometheus/releases/download/v2.51.0/prometheus-2.51.0.linux-amd64.tar.gz
tar xvf prometheus-2.51.0.linux-amd64.tar.gz
cd prometheus-2.51.0.linux-amd64
```

#### **1.2 配置监控目标**

编辑 `prometheus.yml`：

``` yaml
global:
  scrape\_interval: 15s
  evaluation\_interval: 15s

scrape\_configs:
  \- job\_name: "prometheus"    \# 监控自身
    static\_configs:
      \- targets: \["localhost:9090"\]
  \- job\_name: "node"          \# 监控服务器
    static\_configs:
      \- targets: \["<NODE\_IP>:9100"\]  \# 替换为Node Exporter的IP
```

#### **1.3 启动 Prometheus**

```bash
./prometheus \--config.file\=prometheus.yml
```
访问 `http://<PROMETHEUS_IP>:9090` 验证。
    


### **2\. Node Exporter 安装（被监控服务器）**


#### **2.1 下载与启动**
```bash

wget https://github.com/prometheus/node\_exporter/releases/download/v1.7.0/node\_exporter-1.7.0.linux-amd64.tar.gz
tar xvf node\_exporter-1.7.0.linux-amd64.tar.gz
cd node\_exporter-1.7.0.linux-amd64
./node\_exporter
```

默认端口 `9100`，访问 `http://<NODE_IP>:9100/metrics` 查看指标。
    

#### **2.2（可选）配置为系统服务**

创建 `/etc/systemd/system/node_exporter.service`：

``` ini
\[Unit\]
Description\=Node Exporter
\[Service\]
ExecStart\=/path/to/node\_exporter
\[Install\]
WantedBy\=multi-user.target
```

启动服务：
``` bash

sudo systemctl enable \--now node\_exporter
```
* * *


### **3\. Alertmanager 安装**

#### **3.1 下载与解压**

```bash

wget https://github.com/prometheus/alertmanager/releases/download/v0.27.0/alertmanager-0.27.0.linux-amd64.tar.gz
tar xvf alertmanager-0.27.0.linux-amd64.tar.gz
cd alertmanager-0.27.0.linux-amd64
```

#### **3.2 配置邮件告警**

编辑 `alertmanager.yml`：

```yaml

route:
  receiver: 'email-alerts'
receivers:
\- name: 'email-alerts'
  email\_configs:
  \- to: 'your-email@example.com'
    from: 'alertmanager@example.com'
    smarthost: 'smtp.example.com:587'
    auth\_username: 'user@example.com'
    auth\_password: 'your-password'
```

#### **3.3 启动 Alertmanager**

```bash

./alertmanager \--config.file\=alertmanager.yml
```

访问 http://<ALERTMANAGER_IP>:9093 管理告警。

## 4. 配置 Prometheus 告警规则
###  4.1创建告警规则文件
在 Prometheus 目录下创建 alert.rules.yml：
```yaml
groups:
- name: server-alerts
  rules:
  - alert: HighCPUUsage
    expr: 100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[1m])) * 100 > 80
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "High CPU usage on {{ $labels.instance }}"
      description: "CPU usage is {{ $value }}%"
```
### 4.2 修改 prometheus.yml 加载规则
```yaml
rule_files:
  - 'alert.rules.yml'

alerting:
  alertmanagers:
  - static_configs:
    - targets: ['<ALERTMANAGER_IP>:9093']  # 替换为 Alertmanager 的 IP
```
### 4.3 重启 Prometheus
```bash
pkill prometheus
./prometheus --config.file=prometheus.yml
```
## 5. 安装 Grafana
### 5.1 下载与安装
```bash
wget https://dl.grafana.com/oss/release/grafana-10.4.3.linux-amd64.tar.gz
tar xvf grafana-10.4.3.linux-amd64.tar.gz
cd grafana-10.4.3
./bin/grafana-server
```
访问 http://<GRAFANA_IP>:3000（默认账号 admin/admin）。

### 5.2 添加 Prometheus 数据源
1. 进入 Configuration > Data Sources > Add data source。
2. 选择 Prometheus，填写 URL http://<PROMETHEUS_IP>:9090。

### 5.3 导入仪表盘
1. 进入 Dashboards > Import。
2. 输入仪表盘 ID 1860（Node Exporter 官方模板），点击 Load。

## 三、使用示例
### 1. PromQL 查询

    在 Prometheus 的 Web UI 中尝试查询：
    * CPU 使用率：
        ```promql
        100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[1m])) * 100
        ```
    * 内存剩余：
        ```promql
        node_memory_MemAvailable_bytes / 1024 / 1024  # MB
        ```
### 2. 告警测试
触发 HighCPUUsage 告警后，检查邮箱是否收到通知。

###  Grafana 仪表盘

## 优化与扩展
* **长期存储**：集成 Thanos 或 VictoriaMetrics。
* **监控更多服务**：
  * 数据库：mysqld_exporter、postgres_exporter。
  * 高可用：部署多副本 Prometheus + Alertmanager。

## 五、常见问题
### 1. Prometheus 无数据：检查 targets 页面（http://<PROMETHEUS_IP>:9090/targets）是否显示 UP。
### 2. 告警未触发：确认 alert.rules.yml 语法正确，且 Alertmanager 配置无误。
### 3. Grafana 无图表：检查数据源是否连接成功。

通过以上步骤，可以搭建一个完整的 轻量级监控告警系统，覆盖服务器指标采集、可视化、告警全流程！