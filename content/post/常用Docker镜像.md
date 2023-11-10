---
title: "php-msf-docker"
date: 2023-06-01T11:46:01+08:00
draft: false
categories: ["docker"]
tags: ["docker"]
keywords: ["docker","docker-image"]
---


# 常用Docker镜像

## php-msf-docker
```
docker run --privileged --restart=always -it -d --hostname=php-msf  --name=php-msf-docker -p 2202:22 -p 80:80 -p 8000:8000 -p 9501:9501 -v  D:\Develop\Docker\WWW:/php-msf/data/www leanku/php-msf-docker

```

## Rabbitmq
```
docker run -d --name rabbitmq -e RABBITMQ_DEFAULT_USER=guest -e RABBITMQ_DEFAULT_PASS=guest -p 15672:15672 -p 5672:5672 rabbitmq:3.8-management

```

## MySQL
```
docker run -p 13306:3306 --name mysql5.7 -e MYSQL_ROOT_PASSWORD=root -v D:\Develop\Docker\app-mysql:/var/lib/mysql -d mysql:5.7

```

## Redis
```
docker run --name redis -p 6379:6379 -d redis redis-server --appendonly yes

```

## Elasticsearch
```
docker run -d --name es -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch:latest
```


