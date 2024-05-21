---
title: "NGINX负载均衡"
date: 2024-04-20T20:46:01+08:00
draft: false
categories: ["NGINX"]
tags: ["NGINX","负载均衡"]
keywords: ["NGINX","负载均衡"]
---


# NGINX负载均衡

## Nginx是一个强大的开源Web服务器和反向代理服务器，它支持正向代理和反向代理功能。
### 正向代理
正向代理是客户端访问代理服务器去访问目标服务器，并且对目标服务器隐藏了客户端的真实信息（IP等信息）

正向代理的主要用途包括：

* 访问被限制的资源：当某些资源受到网络限制或访问限制时，可以使用正向代理绕过这些限制来获取资源。
* 提高访问速度：代理服务器可以缓存经常请求的资源，从而提高客户端访问资源的速度。
* 突破防火墙：正向代理可以帮助绕过企业或国家防火墙的限制，访问被封锁的网站或资源。
  
### 反向代理
反向代理是指代理服务器接收客户端的请求，然后反向代理将客户端的请求分发给一个或多个目标服务器，最后将响应返回给客户端，对于客户端隐藏了真实的服务端信息

反向代理的主要用途包括：
* 负载均衡：反向代理可以根据一定的算法将请求均匀地分发给后端的多台服务器，从而实现负载均衡，提高系统的并发处理能力和稳定性。
* 缓存静态资源：反向代理可以缓存经常请求的静态资源，减少后端服务器的负载，提高网站的访问速度。
* 安全性和可靠性：反向代理可以作为防火墙和安全设备，提供安全认证、访问控制、DDoS攻击防护等功能。


## Nginx配置文件
```
#user  nobody;   是用来指定 Nginx 进程运行的用户和用户组的配置项。 在 Linux 系统中，各个进程需要以某个用户的身份来运行，以限制权限并提高安全性
worker_processes  1;
 
events {
    worker_connections  1024;
}
 
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
 
    server {
        listen       8080;
        server_name  localhost;
 
        location / {
            root   html;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}
```
第一部分 全局块worker_processes:

    这是 Nginx 服务器并发处理服务的关键配置，worker_processes 值越大，可以支持的并发处理量也越多，但是 会受到硬件、软件等设备的制约。
    
第二部分：events块

     events 块涉及的指令 主要影响 Nginx 服务器与用户的网络连接，常用的设置包括是否开启对多 work process 下的网络连接进行序列化，是否允许同时接收多个网络连接，选取哪种事件驱动模型来处理连接请求，每个 word process 可以同时支持的最大网络连接数等。
上述例子就表示每个 work process 支持的最大连接数为 1024.

第三部分：http块

    http 块也可以包括 http全局块、server 块。

**http全局块**

        http全局块配置的指令包括文件引入、MIME-TYPE 定义、日志自定义、连接超时时间、单链接请求数上限等。

**server 块**

         这块和虚拟主机有密切关系，虚拟主机从用户角度看，和一台独立的硬件主机是完全一样的，该技术的产生是为了 节省互联网服务器硬件成本。



## NGINX负载均衡
NGINX负载均衡主要是对proxy_pass 和 upstream 的配置，需要在http模块下配置upstream模块，然后在server模块下配置proxy_pass模块
```
upstream mall_proxy {
    server 127.0.0.1:8182;
    server 127.0.0.1:8382;
}
```
```
location \ {
    proxy_pass http://mall_proxy;
    proxy_set_header Host $host;
    peoxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $procexy_addr_x_forwarded_for;
}
```

### Nginx常用的实现负载均衡的4种方式

1. 轮询（Round Robin） 
   
    NGINX负载均衡主要是对proxy_pass 和 upstream 的配置，需要在http模块下配置upstream模块，然后在server模块下配置proxy_pass模块 
   ```
        http {
        upstream backend {
            server 192.168.1.101:8080;
            server 192.168.1.102:8080;
            server 192.168.1.103:8080;
        }
    
        server {
            listen 80;
            
            location / {
                proxy_pass http://backend;
            }
        }
    }
   ```
2. IP哈希（IP Hash）
     Nginx根据客户端的IP地址进行哈希运算，并根据计算结果将请求分配给固定的后端服务器。这种算法保证了相同的客户端IP每次请求都会被分配到相同的服务器，适用于需要保持会话状态的应用。
    ```
    http {
        upstream backend {
            ip_hash;
            server 192.168.1.101:8080;
            server 192.168.1.102:8080;
            server 192.168.1.103:8080;
        }
    
        server {
            listen 80;
            
            location / {
                proxy_pass http://backend;
            }
        }
    }
    ```

3. 加权轮询（Weighted Round Robin）：
    Nginx根据每个后端服务器的配置权重将请求分配给服务器。权重越高的服务器，处理的请求就越多。这种方式适用于后端服务器之间配置不同、处理能力不同的情况下。

    ```
    http {
        upstream backend {
            server 192.168.1.101:8080 weight=3;
            server 192.168.1.102:8080 weight=2;
            server 192.168.1.103:8080 weight=1;
        }
    
        server {
            listen 80;
            
            location / {
                proxy_pass http://backend;
            }
        }
    }
    ```

4. 最少连接（Least Connections）：
      Nginx会统计每个后端服务器当前的活动连接数，并将请求分配给活动连接数最少的服务器，以实现负载均衡。这种算法适用于后端服务器配置和处理能力不同、连接持续时间不均衡的场景。
    ```
    http {
        upstream backend {
            least_conn;
            server 192.168.1.101:8080;
            server 192.168.1.102:8080;
            server 192.168.1.103:8080;
        }
    
        server {
            listen 80;
            
            location / {
                proxy_pass http://backend;
            }
        }
    }
    ```


## nginx跨域
当一个请求url的协议、域名、端口号三者之间任意一个与当前页面url不同即为跨域。
```
     #是否允许请求带有验证信息
     add_header Access-Control-Allow-Credentials true;
     #允许跨域访问的域名,可以是一个域的列表，也可以是通配符*
     add_header Access-Control-Allow-Origin  $allow_url;
     #允许脚本访问的返回头
     add_header Access-Control-Allow-Headers 'x-requested-with,content-type,Cache-Control,Pragma,Date,x-timestamp';
     #允许使用的请求方法，以逗号隔开
     add_header Access-Control-Allow-Methods 'POST,GET,OPTIONS,PUT,DELETE';
     #允许自定义的头部，以逗号隔开,大小写不敏感
     add_header Access-Control-Expose-Headers 'WWW-Authenticate,Server-Authorization';
     #P3P支持跨域cookie操作
     add_header P3P 'policyref="/w3c/p3p.xml", CP="NOI DSP PSAa OUR BUS IND ONL UNI COM NAV INT LOC"';
     add_header test  1;
```

## location规则
配置location /wandou可以匹配/wandoudouduo请求，也可以匹配/wandou*/duoduo等等，只要以wandou开头的目录都可以匹配到。而location /wandou/必须精确匹配/wandou/这个目录的请求,不能匹配/wandouduoduo/或/wandou*/duoduo等请求。

第一种：加"/" location /wddd/ { proxy_pass http://127.0.0.1:8080/  (opens new window); } 测试结果，请求被代理跳转到：http://127.0.0.1:8080/index.html 

第二种: 不加"/" location /wddd/ {
 proxy_pass http://127.0.0.1:8080  (opens new window); } 测试结果，请求被代理跳转到：http://127.0.0.1:8080/wddd/index.html 

第三种: 增加目录加"/" location /wddd/ { proxy_pass http://127.0.0.1:8080/sun/; } 测试结果，请求被代理跳转到：http://127.0.0.1:8080/sun/index.html 第四种：增加目录不加"/" location /wddd/ { proxy_pass http://127.0.0.1:8080/sun; } 测试结果，请求被代理跳转到：http://127.0.0.1:8080/sun/index.html 总结 location目录后加"/",只能匹配目录，不加“/”不仅可以匹配目录还对目录进行模糊匹配。而proxy_pass无论加不加“/”,代理跳转地址都直接拼接

|符号|说明|
|:--|:--|
|~|正则匹配，区分大小写|
|~*|正则匹配，不区分大小写|
|^~|普通字符匹配，如果该选项匹配，则，只匹配改选项，不再向下匹配其他选项|
|=|普通字符匹配，精确匹配|
|@|定义一个命名的 location，用于内部定向，例如 error_page，try_files|


## 全局变量
```
$bytes_sent             发送给客户端的字节数
$connection             连接序列号
$connection_requests    当前通过连接发出的请求数量
$content_length         “Content-Length” 请求头字段
$content_type           “Content-Type” 请求头字段
$cookie_name            cookie名称
$document_root          当前请求的文档根目录或别名
$uri                    请求中的当前URI(不带请求参数，参数位于$args)，可以不同于浏览器传递的$request_uri的值，它可以通过内部重定向，或者使用index指令进行修改，$uri不包含主机名，如”/foo/bar.html”。
$document_uri           同 $uri
$host                   优先级如下：HTTP请求行的主机名>”HOST”请求头字段>符合请求的服务器名
$hostname               主机名
$http_name              匹配任意请求头字段； 变量名中的后半部分“name”可以替换成任意请求头字段，如在配置文件中需要获取http请求头：“Accept-Language”，那么将“－”替换为下划线，大写字母替换为小写，形如：$http_accept_language即可。
$https                  如果开启了SSL安全模式，值为“on”，否则为空字符串。
$is_args                如果请求中有参数，值为“?”，否则为空字符串。
$limit_rate             用于设置响应的速度限制
$nginx_version          nginx版本
$pid                    工作进程的PID
$pipe                   如果请求来自管道通信，值为“p”，否则为“.” (1.3.12, 1.2.7)
$proxy_protocol_addr    获取代理访问服务器的客户端地址，如果是直接访问，该值为空字符串。
$realpath_root          当前请求的文档根目录或别名的真实路径，会将所有符号连接转换为真实路径。
$msec                   以秒为单位的时间，日志写入时的毫秒分辨率
$request_length         请求长度（包括请求行，标题和请求主体）
$request_method         HTTP请求方法，通常为“GET”或“POST”
$request_time           处理客户端请求使用的时间;从读取客户端的第一个字节开始计时。
$request_uri            这个变量等于包含一些客户端请求参数的原始URI，它无法修改，请查看$uri更改或重写URI，不包含主机名，例如：”/cnphp/test.php?arg=freemouse”。
$request_time           以毫秒分辨率请求处理时间，以秒为单位; 从客户端读取第一个字节之间的时间并在最后一个字节发送到客户端后进行日志写入
$status                 响应状态码
$time_local             本地时间采用通用日志格式
$arg_name               请求中的的参数名，即“?”后面的arg_name=arg_value形式的arg_name
$args                   请求中的参数值
$binary_remote_addr     客户端地址的二进制形式, 固定长度为4个字节
$body_bytes_sent        传输给客户端的字节数，响应头不计算在内；这个变量和Apache的mod_log_config模块中的“%B”参数保持兼容
$remote_addr            客户端地址
$remote_port            客户端端口
$remote_user            用于HTTP基础认证服务的用户名
$request                客户端请求地址
$request_body           客户端的请求主体，此变量可在location中使用，将请求主体通过proxy_pass, fastcgi_pass, uwsgi_pass, 和 scgi_pass传递给下一级的代理服务器。
$request_body_file      将客户端请求主体保存在临时文件中。文件处理结束后，此文件需删除。如果需要之一开启此功能，需要设置client_body_in_file_only。如果将次文件传递给后端的代理服务器，需要禁用request body，即设置proxy_pass_request_body off，fastcgi_pass_request_body off, uwsgi_pass_request_body off, or scgi_pass_request_body off 。
$request_completion     如果请求成功，值为”OK”，如果请求未完成或者请求不是一个范围请求的最后一部分，则为空。
$request_filename       当前连接请求的文件路径，由root或alias指令与URI请求生成。
$scheme                 请求使用的Web协议, “http” 或 “https”
$sent_http_name         可以设置任意http响应头字段； 变量名中的后半部分“name”可以替换成任意响应头字段，如需要设置响应头Content-length，那么将“－”替换为下划线，大写字母替换为小写，形如：$sent_http_content_length 4096即可。
$server_addr            服务器端地址，需要注意的是：为了避免访问linux系统内核，应将ip地址提前设置在配置文件中。
$server_name            服务器名，域名
$server_port            服务器端口
$server_protocol        服务器的HTTP版本, 通常为 “HTTP/1.0” 或 “HTTP/1.1”
$tcpinfo_rtt, $tcpinfo_rttvar, $tcpinfo_snd_cwnd, $tcpinfo_rcv_space 客户端TCP连接的具体信息
```
