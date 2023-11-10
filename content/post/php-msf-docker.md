---
title: "php-msf-docker"
date: 2023-03-11T11:46:01+08:00
draft: false
categories: ["docker"]
tags: ["docker"]
keywords: ["docker","php-msf-docker"]
---


# php-msf-docker
基于[centos:centos7.9.2009](https://hub.docker.com/layers/library/centos/centos7.9.2009/images/sha256-dead07b4d8ed7e29e98de0f4504d87e8880d4347859d839686a31da35a3b532f?context=explore)镜像制作的php开发环境镜像

#### 主要包含：
* PHP-8.1.14
* redis-5.3.7
* swoole-src-5.0.2
* nginx-1.21.5
* supervisor
* sshd

**GitHub地址 :** [https://hub.docker.com/r/leanku/php-msf-docker](https://github.com/leanku/php-msf-docker)

**DockerHub地址 :** [https://hub.docker.com/r/leanku/php-msf-docker](https://hub.docker.com/r/leanku/php-msf-docker)   

### 包含扩展
```bash
bcmath,Core,ctype,curl,date,dom,exif,fileinfo,filter,ftp,gd,gettext,hash,iconv,intl,json,libxml,mbstring,mysqli,mysqlnd,openssl,pcntl,pcre,PDO,pdo_mysql,pdo_pgsql,pdo_sqlite,pgsql,Phar,posix,redis,Reflection,session,shmop,SimpleXML,soap,sockets,sodium,SPL,sqlite3,standard,sysvsem,tokenizer,xml,xmlreader,xmlwriter,xsl,zip,zlib,swoole
``` 

------

### 运行示例
```
docker run --privileged --restart=always -it -d \
--hostname=php-msf --name=php-msf-docker \
-p 22:22 -p 80:80 -p 3306:3306 -p 8000:8000 -p 9501:9501 \
-v /c/docker/www:/php-msf/data/www \
leanku/php-msf-docker
```
------

### SSH  

默认用户：super   密码：123456

``` shell
ssh super@127.0.0.1
```
