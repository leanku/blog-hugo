---
title: "docker删除none镜像"
date: 2023-03-11T20:46:01+08:00
draft: false
categories: ["docker"]
tags: ["docker"]
keywords: ["docker"]
---

# docker删除none镜像

1、使用git bash进入到docker文件夹

2、查询所有的none镜像

``` shell
docker images  | grep none
```

3、查询所有的none镜像的id

``` shell
docker images  | grep none | awk '{print $3}'
```

4、删除所有的none镜像

``` shell
docker images  | grep none | awk '{print $3}' | xargs docker rmi
```

## docker none镜像说明

### 一、有效的 none 镜像
Docker文件系统的组成，docker镜像是由很多 layers组成的，每个 layer之间有父子关系，所有的docker文件系统层默认都存储在/var/lib/docker/graph目录下，docker称之为图层数据库。

最后做一个总结< none>:< none> 镜像是一种中间镜像，我们可以使用docker images -a来看到，他们不会造成硬盘空间占用的问题（因为这是镜像的父层，必须存在的），但是会给我们的判断带来迷惑。

### 二、无效的 none 镜像

另一种类型的 < none>:< none> 镜像是dangling images ，这种类型会造成磁盘空间占用问题。

像Java和Golang这种编程语言都有一个内存区，这个内存区不会关联任何的代码。这些语言的垃圾回收系统优先回收这块区域的空间，将他返回给堆内存，所以这块内存区对于之后的内存分配是有用的

docker的悬挂(dangling)文件系统与上面的原理类似，他是没有被使用到的并且不会关联任何镜像，因此我们需要一种机制去清理这些悬空镜像。

我们在上文已经提到了有效的< none>镜像，他们是一种中间层，那无效的< none>镜像又是怎么出现的？这些 dangling镜像主要是我们触发 docker build 和 docker pull命令产生的。

使用下面的命令可以清理

``` shell
docker rmi $(docker images -f "dangling=true" -q)
```