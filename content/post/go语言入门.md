---
title: "Go"
date: 2024-02-18T20:46:01+08:00
draft: false
categories: ["Go"]
tags: ["Go"]
keywords: ["Go"]
---


# Go语言
    Go（又称 Golang）是 Google 的 Robert Griesemer，Rob Pike 及 Ken Thompson 开发的一种静态强类型、编译型语言。Go 语言语法与 C 相近，但功能上有：内存安全，GC（垃圾回收），结构形态及 CSP-style 并发计算。

## 安装
下载对应系统的安装包进行安装  
[下载地址](https://studygolang.com/dl)  
安装成功后命令行输入 ```go``` 正常会出现 Go is a tool for managing Go source code. 等一些提示， 输入 ```go version``` 可查看当前安装的go语言版本

## 设置
使用 go env 命令查看配置

```export GO111MODULE=on```    # 设置环境变量，windows的同学使用SET GO111MODULE=on  这里on必须是小写的，不是大写ON

```go env -w GO111MODULE=on```  # 重新向go env写入正确的值

```go env -w GOPROXY=https://goproxy.cn,direct```   #设置Go 模块代理 [官方文档](https://goproxy.cn/)  

再次使用 ```go env``` 命令验证配置


## 编辑器配置

### GoLand
下载 GoLand 安装go,File Watchers 插件

    goimports工具是Go官方提供的一种工具，它能够为我们自动格式化 Go 语言代码并对所有引入的包进行管理，包括自动增删依赖的包引用、将依赖包按字母序排序并分类。
命令行执行 ``` go install golang.org/x/tools/cmd/goimports@latest``` 安装goimports  

到GoLand 中Settings>Tools>File Watchers 添加goimports

新项目编辑器会自动生成.mod 文件，也可手动生成命令 ```go mod init 项目名```

运行项目在GoLand中会有绿色箭头，也可命令执行 ```go run 文件名```


## 依赖管理 Go Modules（Go Mod）、GOPATH
### GOPATH 模式（GO111MODULE=off）
    Go 编译器从不使用 Go Mod。而会查找 vendor 目录和 GOPATH 以查找依赖项。

    GOPATH指定外部Go语言代码开发工作区目录, 在 Go 1.5 版本之前，所有的依赖包都是存放在 GOPATH下，没有版本控制。这种方式的最大的弊端就是无法实现包的多版本控制，比如项目 A 和项目 B 依赖于不同版本的 package，如果 package 没有做到完全的向前兼容，可能会导致一些问题。从Go 1.8版本开始Go开发包在安装完成后会为GOPATH设置一个默认目录，并且在Go 1.14及之后的版本中启用了Go Module模式之后，不一定非要将代码写到GOPATH目录下，所以也就不需要我们再自己配置GOPATH了使用默认的即可.

    1.5 版本推出了 vendor 机制，所谓 vendor 机制，就是每个项目的根目录下可以有一个 vendor 目录，里面存放了该项目的依赖的 package。go build 的时候会先去 vendor 目录查找依赖，如果没有找到会再去 GOPATH 目录下查找。

### Go Modules 模式（ GO111MODULE=on）
    Go 编译器只使用 Go Mod，GOPATH不再作为导入目录，但它还是会把下载的依赖储存在 GOPATH/pkg/mod 中，也会把 go install 命令的结果放在 GOPATH/bin 中。

    Go 1.11 版本推出 modules，简称 mod，go mod的推出可以使我们更容易管理项目中所需要的模块。go mod 不再依靠 $GOPATH，使得它可以脱离 GOPATH 来创建项目。也就是你可以在你电脑的任意位置创建一个文件夹作为项目目录，然后使用 go mod 命令对目录进行初始化

```
go get [@版本], go mod tidy  #更新依赖
go mod init, go build ./...  #将旧项目迁移到go mod 
```

## 目录整理
含有main函数的文件必须在自己单独的目录

## 命令
go env用于打印Go语言的环境信息。

go run命令可以编译并运行命令源码文件。

go get可以根据要求和实际情况从互联网上下载或更新指定的代码包及其依赖包，并对它们进行编译和安装。

go build命令用于编译我们指定的源码文件或代码包以及它们的依赖包。

go install用于编译并安装指定的代码包及它们的依赖包。

go clean命令会删除掉执行其它命令时产生的一些文件和目录。

go doc命令可以打印附于Go语言程序实体上的文档。我们可以通过把程序实体的标识符作为该命令的参数来达到查看其文档的目的。

go test命令用于对Go语言编写的程序进行测试。

go list命令的作用是列出指定的代码包的信息。

go fix会把指定代码包的所有Go语言源码文件中的旧版本代码修正为新版本的代码。

go vet是一个用于检查Go语言源码中静态错误的简单工具。

go tool pprof命令来交互式的访问概要文件的内容。



## 基础语法

TODO

#### 变量
TODO

##### 内建变量类型：
* bool, string
* (u)int,(u)int8,(u)int16,(u)int32,(u)int64,uintptr     加u代表为无符号
* byte,rune
* float32,float64,complex64,complex128
[官方文档](http://docscn.studygolang.com/doc/)  