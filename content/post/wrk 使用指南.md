---
title: "wrk 使用指南"
date: 2025-06-30T23:46:01+08:00
draft: false
categories: ["DevOps"]
tags: ["wrk"]
keywords: ["wrk"]
---

# wrk 使用指南
## 一、介绍
wrk 是一款现代化的 HTTP 基准测试工具，使用 C 语言编写，基于事件通知机制（如 epoll, kqueue），能够产生巨大的负载。相比 ab(apache benchmark)，wrk 具有以下优势：

-   支持多线程 + 协程模式，能更好地利用多核 CPU
    
-   支持 LuaJIT 脚本扩展，可自定义请求生成和结果处理
    
-   性能更高，单机可轻松产生数万 QPS
    
-   提供更详细的统计信息（延迟分布等）



## 二、安装 wrk
Linux 系统安装
```bash
# Ubuntu/Debian
sudo apt install wrk -y

# CentOS/RHEL
sudo yum install wrk -y

# 或从源码编译安装
git clone https://github.com/wg/wrk.git
cd wrk
make
sudo cp wrk /usr/local/bin/
```
macos
```bash
brew install wrk
```

## 三、基础使用方法
### 1.  基本命令格式
```bash
wrk <选项> <测试URL>
```
### 2. 常用选项说明
|选项|说明|示例值|
|:--|:--|:--|
|-t|使用的线程数|12 (建议设置为CPU核心数的2-4倍)|
|-c|保持打开的连接数|100|
|-d|测试持续时间|30s (30秒), 2m (2分钟)|
|-s|指定Lua脚本|post.lua
|-H|添加HTTP头|"User-Agent: wrk"|
|--latency|打印延迟统计|无参数|
|--timeout|超时设置|10s|

### 3. 基本测试示例
```bash
# 简单测试：12线程，100连接，持续30秒
wrk -t12 -c100 -d30s http://localhost:8080/

# 带延迟统计的输出
wrk -t12 -c100 -d30s --latency http://localhost:8080/api

# 带自定义HTTP头
wrk -t12 -c100 -d30s -H "Authorization: Bearer token123" http://localhost:8080/api
```

## 四、高级功能：使用 Lua 脚本
wrk 的强大之处在于支持通过 Lua 脚本自定义请求。
### 1. 脚本基本结构
```lua
-- 初始化阶段
function setup(thread)
  -- 每个线程初始化时调用
end

-- 请求生成
function request()
  -- 返回一个HTTP请求
end

-- 响应处理
function response(status, headers, body)
  -- 对响应进行处理
end
```

### 2. 常用脚本示例
POST 请求示例 (post.lua)
```lua
wrk.method = "POST"
wrk.headers["Content-Type"] = "application/json"
wrk.body = '{"username":"test","password":"123456"}'
```
动态参数请求

```lua
counter = 1

request = function()
   path = "/api/v1/items/" .. counter
   counter = counter + 1
   return wrk.format("GET", path)
end
```

认证请求
```lua
token = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."

wrk.headers["Authorization"] = "Bearer " .. token
wrk.headers["Content-Type"] = "application/json"
```

### 3. 使用脚本运行测试
```bash
wrk -t4 -c100 -d10s -s post.lua http://localhost:8080/api/login
```

## 五、结果解读
典型输出示例：
```text
Running 30s test @ http://localhost:8080/
  12 threads and 100 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    54.32ms   15.22ms 212.34ms   75.23%
    Req/Sec   153.45     32.67   250.00     68.50%
  Latency Distribution
     50%   51.23ms
     75%   62.45ms
     90%   78.90ms
     99%  105.67ms
  54876 requests in 30.10s, 42.36MB read
Requests/sec:   1823.08
Transfer/sec:      1.41MB
```
关键指标说明：
-   **Latency (延迟)**
    
    -   Avg: 平均响应时间
        
    -   Stdev: 标准差
        
    -   Max: 最大响应时间
        
    -   +/- Stdev: 数据分布
        
    -   分布线: 50%, 75%, 90%, 99% 百分位响应时间
        
-   **Req/Sec (每秒请求数)**
    
    -   每个线程每秒完成的请求数统计
        
-   **汇总信息**
    
    -   总请求数
        
    -   总测试时间
        
    -   总数据传输量
        
    -   Requests/sec: 系统QPS (重要指标)
        
    -   Transfer/sec: 吞吐量

## 六、实用技巧
### 1. 渐进式压力测试
```bash
for c in 50 100 200 500 1000; do
echo "Testing with $c connections"
wrk -t12 -c$c -d30s http://localhost:8080/
echo ""
done
```

### 2. 保持测试一致性
```bash
# 使用固定随机种子
math.randomseed(os.time())

request = function()
id = math.random(1, 10000)
path = "/api/items/" .. id
return wrk.format("GET", path)
end
```

### 3. 结果输出到文件
```bash
wrk -t12 -c100 -d30s http://localhost:8080/ > result.txt
```

### 4. 测试HTTPS接口
```bash
wrk -t12 -c100 -d30s https://example.com/api
```
## 七、注意事项
-   **线程数设置**：建议从 CPU 核心数开始，逐步增加
    
-   **连接数设置**：不要超过服务器最大连接数限制
    
-   **测试时间**：至少 30 秒以获得稳定结果
    
-   **结果分析**：关注 99% 延迟而不仅是平均值
    
-   **网络影响**：最好在同一内网测试，排除网络因素
    
-   **预热**：正式测试前先进行 1-2 分钟预热

## 八、与其他工具对比
| 特性 | wrk | ab (apache benchmark) | JMeter |
| :--|:--|:--|:--|
| 性能 |  非常高 | 中等 | 低 |
| 学习曲线 | 低 | 非常低 | 高 | 
| 脚本支持 | Lua | 无 | Java/Groovy | 
| 分布式测试 | 不支持 | 不支持 | 支持 | 
| 结果详细度 | 中等 | 简单 | 非常详细 | 

