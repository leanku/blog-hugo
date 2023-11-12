---
title: "Consul注册中心"
date: 2023-06-01T11:46:01+08:00
draft: false
categories: ["微服务"]
tags: ["consul"]
keywords: ["consul","注册中心","微服务"]
---


# Consul注册中心

## CAP原理
* 一致性（Consistency） 所有节点在同一时间具有相同的数据
* 可用性（Availablility） 保证每个请求不管或者失败都有响应
* 分区容错（Partition tolerance） 系统中任意数据的丢失或失败不会影响系统的继续运作

## Consul 介绍
* 方便部署
* 采用Raft算法实现，有服务发现，key/value存储，可以做配置中心使用。有健康检查，同事提供了web管理页面

## Consul角色
* -dev 开发环境下的启动命令，提供基本的服务
* -client 客户端，无状态的，将http和dns请求转发到服务端集群
* -server 服务端，保存配置信息，可以搭建高可用的集群

## Consul 内部端口
|端口|说明|
|----|----|
|TCP/8300 | 8300端口用于服务器节点。客户端通过该端口RPC协议调用服务端接口。|
|TCP/UDP/8301 |8301端口用于单个数据中心所有节点之间的相互通信，即对LAN池信息的同步。它使得整个数据中心能够自动发现服务器地址，分布式检测节点故障，事件广播|
|TCP/UDP/8302| 8302端口用于单个或多个数据中心之间的服务器节点的信息同步。即对WAN池信息的同步。它针对互联网的高延迟进行了优化，能够实现跨数据中心请求。|
|8500| 8500端口基于HTTP协议，用于API接口或WEB UI访问。|
|8600|8600端口作为DNS服务器，它使得我们可以通过节点名查询节点的信息。|

## Consul 工作原理
![consul工作原理](http://resources.leanku.com/consul.png)



## Consul 安装
[下载地址](https://developer.hashicorp.com/consul/install)

安装完成后，命令行输入consul 检查 consul 是否可用

Consul支持web ui界面。UI可用于查看所有服务和节点，查看所有运行状况检查及其当前状态，以及读取和设置键/值数据。 用户界面自动支持多数据中心。要设置自带的UI，请使用-ui参数启动Consul代理：
```
consul agent -ui
```
UI可以在与HTTP API相同的端口上的/ui路径中使用。 默认情况下，这是http://localhost:8500/ui。

可以在[这里](https://cloud.tencent.com/developer/tools/blog-entry?target=http%3A%2F%2Fdemo.consul.io%2F%3F_ga%3D2.168770787.357465016.1511587452-891659360.1511587452)查看Consul Web UI的现场演示。


## 注册个服务

使用HTTP API 注册个服务，使用[接口API](https://www.consul.io/api/agent/service.html API)调用

调用 http://consul:8500/v1/agent/service/register PUT 注册一个服务。
```
#request body:
{
  "ID": "userServiceId", //服务id
  "Name": "userService", //服务名
  "Tags": [              //服务的tag，自定义，可以根据这个tag来区分同一个服务名的服务
    "primary",
    "v1"
  ],
  "Address": "127.0.0.1",//服务注册到consul的IP，服务发现，发现的就是这个IP
  "Port": 8000,          //服务注册consul的PORT，发现的就是这个PORT
  "EnableTagOverride": false,
  "Check": {             //健康检查部分
    "DeregisterCriticalServiceAfter": "90m",
    "HTTP": "http://www.baidu.com", //指定健康检查的URL，调用后只要返回20X，consul都认为是健康的
    "Interval": "10s"   //健康检查间隔时间，每隔10s，调用一次上面的URL
  }
}


## 使用curl调用
curl http://172.17.0.4:8500/v1/agent/service/register -X PUT -i -H "Content-Type:application/json" -d '{
  "ID": "userServiceId",  
  "Name": "userService",
  "Tags": [
    "primary",
    "v1"
  ],
  "Address": "127.0.0.1",
  "Port": 8000,
  "EnableTagOverride": false,
  "Check": {
    "DeregisterCriticalServiceAfter": "90m",
    "HTTP": "http://www.baidu.com",
    "Interval": "10s"
  }
}'
```

## 发现服务
刚刚注册了名为userService的服务，我们现在发现（查询）下这个服务
```
# 使用curl调用
curl http://172.17.0.4:8500/v1/catalog/service/userService


# 返回的响应：
[
    {
        "Address": "172.17.0.4",
        "CreateIndex": 880,
        "ID": "e6e9a8cb-c47e-4be9-b13e-a24a1582e825",
        "ModifyIndex": 880,
        "Node": "node3",
        "NodeMeta": {},
        "ServiceAddress": "127.0.0.1",
        "ServiceEnableTagOverride": false,
        "ServiceID": "userServiceId",
        "ServiceName": "userService",
        "ServicePort": 8000,
        "ServiceTags": [
            "primary",
            "v1"
        ],
        "TaggedAddresses": {
            "lan": "172.17.0.4",
            "wan": "172.17.0.4"
        }
    }
]
```



