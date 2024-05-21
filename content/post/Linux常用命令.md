---
title: "Linux常用命令"
date: 2024-03-18T20:46:01+08:00
draft: false
categories: ["Linux"]
tags: ["Linux"]
keywords: ["Linux"]
---


# Linux基本简介
    Linux 是一个基于Linux 内核的开源类Unix 操作系统，Linus Torvalds于 1991 年 9 月 17 日首次发布的操作系统内核。Linux 通常打包为Linux 发行版。

## Linux文件系统
* Linux一切皆文件
* 只有一个顶级目录，不像windows分C盘、D盘、E盘

|Linux|	含义|	windows|
|:--|:--|:--|
|/bin	|所有用户可用的基本命令存放的位置	|windows没有固定的命令存放目录
|/sbin	|需要管理员权限才能使用的命令	
|/boot	|linux系统启动的时候需要加载和使用的文件	
|/dev	|外设连接linux后，对应的文件存放的位置	|类似Windows中的U盘，光盘的符号文件。
|/etc	|存放系统或者安装的程序的配置文件,注册服务等	|类似windows中的注册表，
|/home	|家目录，linux中每新建一个用户，会自动在home中为该用户分配一个文件夹	|类似windows中的"我的文档"，每个用户有自己的目录。
|/root	|root账户的家目录，仅供root账户使用	|类似windows中的Administrator账户的"我的文档"
|/lib	|linux的命令和系统启动，需要使用一些公共的依赖，放在lib中，类似我们开发的代码执行需要引入的jdk的jar	
|/usr	|很多系统软件的默认安装路径	|类似windows中的C盘下的Program Files目录。
|/var	|系统和程序运行产生的日志文件和缓存文件放在这里

## Linux常用命令 
命令格式 ：命令 [-选项] [参数]

    例 ： ls -la /etc
### 文件管理
|命令|	解释|	参数|   示例|
|:--|:--|:--|:--|
|ls|列出目录的内容| -a 显示所有文件，包括隐藏文件; -l详细信息显示; -d 查看目录属性|ls -la|
|cd|切换工作目录|cd  ..|
|pwd|显示当前路径|  |
|cd|切换工作目录|   |cd  ..|
|mkdir |创建目录|  -p  递归创建 | mkdir -p /tmp/zhang/test  |
|rmdir |删除目录|   |  rmdir /tmp/zhang/test |
|rm |删除文件|-r  删除目录;  -f  强制执行   | rm -rf  /tmp/zhang/test2  |
|cp | 拷贝文件|-r  复制目录;    -p  保留文件属性   | cp  -r /tmp/zhang/test1  /root  |
|mv |移动文件|   |  mv 文件A 文件B |
|md5sum |获取文件的md5指纹|   | md5sum 文件名  |


### 文本内容(查看|处理)命令
|命令|	解释|	参数|   示例|
|:--|:--|:--|:--|
|touch|创建文件|    |touch test|
|cat        | 显示文件内容     |  -n  显示行号  | cat  /etc/issue     |
|more       |分屏显示     | (空格) 或f  翻页; (Enter) 换行; q或Q 退出    |   |
|less       |分屏显示     |    |   |
|head       | 取首n行    |  -n 指定行数   |head -n 20 /etc/services   |
|tail        |取尾n行     |-n 指定行数; -f  动态显示文件末尾内容     | tail -n 18 /etc/services  |


### 文件搜索命令
|命令|	解释|	参数|   示例|
|:--|:--|:--|:--|
|find|查找文件|  [搜索范围]  [匹配条件]  |find / -name "passwd"|
|locate       |文件资料库查找     |    | locate inittab  |
|which       | 查看命令位置     |    | which ls  |
|whereis        |  查看命令位置    |    |   |
|grep       |文本搜索     |-i  不区分大小写;    -v 排除指定字串    | grep  mysql  /root/install.log  |


### 文件链接
|命令|	解释|	参数|   示例|
|:--|:--|:--|:--|
|ln |文件链接   |-s  创建软链接     |ln  /etc/issue  /tmp/issue.hard |


### 权限管理命令
**用户组**

* 创建组  `groupadd 组名`

* 删除组  `groupdel 组名`

* 找系统中的组    `cat /etc/group | grep -n “组名”`

  说明：系统每个组信息都会被存放在/etc/group的文件中

**用户**

* 创建用户    `useradd -g 组名 用户名`

* 设置密码     `passwd 用户名`
* 
* 查看登录用户：   `who`

 查找系统账户

  说明：系统每个用户信息保存在`/etc/passwd`文件中

* 切换用户    `su 用户名`

* 删除用户    `userdel -r 用户名`

**权限管理命令**

|权限字母|	含义|	对文件|	代表命令|	对文件夹|	代表命令|
|:--|:--|:--|:--|:--|:--|
|R®|	读|	查看文件内容和复制文件|	more cat less cp head tail	|查看文件夹下的文件|	ls|
|W(w)|	写|	编辑文件|	vi	|在文件夹内创建和删除文件|	rm touch|
|X(x)|	执行|	执行该文件(执行必须具备r权限)|	-	|切换到文件夹|	cd|


* 修改文件权限 chmod 命令 语法：chmod  [{ugoa}{+-=}{rwx}] [文件或目录]   
* 修改文件所有者：chown 命令  语法：chown  [用户] [文件或目录]
* 修改文件所属组：chgrp 命令  语法：chgrp [用户组] [文件或目录]
* 默认权限：umask 命令   umask [-S]   ,-S   以rwx形式显示新建文件缺省权限 

### 压缩解压缩命令
|命令|	解释|	参数|   示例|
|:--|:--|:--|:--|
|gzip |压缩,压缩后文件格式：.gz|    |gzip text01|
|gunzip       |  解压缩    |    |  gunzip text01.gz |
| zip        |压缩文件或目录压缩后文件格式：.zip     |  -r    压缩目录   |zip  text01.zip  text01    |
| unzip      | 解压    |    | unzip text01.zip  |
|tar       |     | -c    打包；-v    显示详细信息；-f     指定文件名；-z     打包同时压缩    |   |

tar压缩语法：tar -zcvf 压缩后文件名 被压缩文件

解压缩语法 tar -zxvf 压缩文件名

待补充..
