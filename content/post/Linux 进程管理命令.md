---
title: "Linux 进程管理命令"
date: 2025-02-17T22:33:45+08:00
draft: false
categories: ["Linux"]
tags: ["Linux"]
keywords: ["Linux","grep"]
---



## Linux ps命令

> ps 命令是 Process Status 的缩写，是一个命令行实用程序，用于显示或查看与Linux系统中运行的进程相关的信息。
> 
> 命令原理：ps 是通过读取虚拟文件：/proc 拿到进程数据的，不需要给 ps 设置任何的权限就可以运行。

**在Linux中，每个进程都有多个ID来关联它，包括：**

1. Process ID (PID)，即进程ID

    这是标识进程的任意数字。每个进程都有一个唯一的ID，但是在进程退出并且父进程检索了退出状态之后，进程ID将被释放，供新进程重用。

2. Parent Process ID (PPID)，即父进程ID

    如果父进程在子进程退出之前退出，则子进程的PPID将更改为另一个进程。

3. Process Group ID (PGID)

    这是进程组leader的PID。如果PID == PGID，则此进程是进程组领导。

4. Session ID (SID)

    这是会话领导者的PID。如果PID == SID，则此进程为会话领导进程。

    会话和进程组只是将一些相关的进程作为一个单元来对待的方法。一个进程组的所有成员总是属于同一个会话，但是一个会话可以有多个进程组。

    通常，shell将是会话领导者，该shell执行的每个管道将是一个进程组。这是为了在shell退出时很容易杀死它的子进程。


grep -option（参数） ‘word’（关键词） file（文本文件）；

### ps命令选项：

***简单筛选***

a ：选择所有进程（BSD-Style）。

-A ：选择所有进程，与 -e 等同（标准格式）。

-a ：选择除 session 领导者和没有与终端关联的进程之外的所有进程。

-d ：选择除了 session leader 的所有进程。

--deselect ：选择除满足指定条件之外的所有进程，即反选，与 -N 等同。

-e ：选择所有进程，与 -A 等同。

-N ：与 --deselect 等同。

T ：选择与该终端关联的所有进程，和不带参数的 t 选项等同。

r ：仅选择运行中的进程。

x ：与 a 一起配合使用。

***通过列表筛选进程***

-C [cmdlist] ： 通过命令的名称筛选，以前的 procps 和内核版本会截断这个命令名为15个字符，在新版本这个限制已剔除，如果想通过模糊匹配的方式，则此法不通。

-G [grplist] ：通过真实的组ID或组名来筛选，即创建此进程的用户所属的组。

-g [grplist] ：通过session或有效的组名来筛选，仅仅当组名也指定时，组ID才能生效。

--Group [grplist] ：与 -G 等同。

--group [grplist] ：与 -g 等同。

p [pidlist] ：通过进程id筛选。

-p [pidlist] ：与 p 等同。

--pid [pidlist] ：与 p 等同。

--ppid [pidlist] ：通过父进程id筛选子进程。

q [pidlist] ：通过进程id筛选（快速模式）。

-q [pidlist] ：与 q 等同。

--quick-pid [pidlist] ：与 q 等同。

-s [sesslist] ：通过sessionID筛选。

--sid [sesslist] ：与 -s 等同。

t [ttylist] ：通过tty（终端）筛选，ttylist 可以为空，与 -t 和 --tty 几乎相同。

-t [ttylist] ：通过tty（终端）筛选，tty 可以使用多种形式，如：/dev/ttyS1、ttyS1、S1，[ttylist]如果是 - 则筛选没有关联到任何终端的进程。

--tty [ttylist] ：与 -t 和 t 等同。

U [userlist] ：通过有效的用户ID或用户名来筛选。

-U [userlist] ：通过真实的用户ID或用户名来筛选，创建此进程的用户即为真实的用户。

-u [userlist] ：与 U 等同。

--User [userlist] ：与 -U 等同。

--user [userlist] ：与 -u 和 U 等同。


***输出格式控制***

-c ：在指定了 -l 格式时，显示不同的调度程序的信息。

--context ：对于 SELinux 显示安全的上下文格式。

-f ：列出完整的格式，一些合并使用的选项，如接 -L，NLWP(线程数)，LWP(线程id) 两个字段会显示出来。

--format [format] ：用户可以指定格式来显示，与 -o 和 o 等同。

j ：BSD 任务控制格式。

-j ：任务格式。

l ：BSD 长格式。

-l ：长格式，长格式即显示更多的列。

-M ：添加一个安全数据的列，与 Z 等同。

O [format] ：自定义显示的列，但会显示一些公共的预定义列，预 -O 等同，此为 BSD 风格。

-O [format] ：自定义显示的列，但会显示一些公共的预定义列，输入 -O 等于 -o pid,format,state,tname,time,command，或者 -o pid,format,tname,time,cmd，与 O [format] 等同。

o [format] ：指定用户定义的格式，与 -o 和 --format 等同。

-o [format] ：

###  ps命令示例

``` shell
# 通过 -o 自定义指定列头
ps -e -o pid,user,comm

# 通过进程id筛选
ps -p 1234

# 通过用户筛选
ps -u root

# 处理僵尸进程
ps -A -ostat,ppid,pid,cmd | grep -e '^[Zz]'

# 显示当前shell匹配的进程信息
ps -p $$

# 列出所有进程并带有完整格式
ps -ef | less

# 通过进程名筛选
ps -C systemd

# 使用BSD风格显示所有进程
ps aux | less

# 组合 grep 来过滤
ps -ef | grep systemd

```

## Linux pstree命令

pstree命令能清晰的表达程序之间的层级相互关系
在 centos/redhat 系列Linux中需要单独安装
yum install psmisc -y 

## Linux pgrep命令
通过程序的名字去查询相关进程，一般用来判断进程是否存活
pgrep nginx #判断NGINX进程是否存在

## Linux kill命令
发送指定的信号到相应进程。不指定型号将发送SIGTERM（15）终止指定进程。如果任无法终止该程序可用“-KILL” 参数，其发送的信号为SIGKILL(9) ，将强制结束进程，使用ps命令或者jobs 命令可以查看进程号。root用户将影响用户的进程，非root用户只能影响自己的进程。

### 命令参数：
-l 信号，若果不加信号的编号参数，则使用“-l”参数会列出全部的信号名称

-a 当处理当前进程时，不限制命令名和进程号的对应关系

-p 指定kill 命令只打印相关进程的进程号，而不发送任何信号

-s 指定发送信号

-u 指定用户

### 常用信号
|信号名称|信号编号|描述|
|:--|:--|:--|
|HUP  |1|终端断开|
|INT  |2|中断（Ctrl+c）|
|QUIT  |3|退出（Ctrl+\）|
|KILL  |9|强制终止|
|TERM  |15|正常终止|
|STOP  |19,20|暂停进程|
|CONT  |18|恢复暂停的进程|
|USR1-31  |10-16,30,31|用户自定义信号，可根据需要进行扩展|

### 使用实例：
``` shell
# 列出所有信号名称
kill -l

# 终止进程
kill <PID> 或
kill -s <信号> <PID>

# 僵尸进程是指进程已经结束了，但它的父进程还没有处理掉它的信息。可以使用kill命令向该进程发送信号，让父进程处理它。比如：
kill -s TERM <PID>

# 当我们想要终止或重新启动一个定时任务时，可以使用kill命令。比如：
kill -s HUP <PID>  # 重启定时任务
kill -s TERM <PID>  # 终止定时任务

#当我们需要释放系统资源时，可以使用kill命令终止不需要的进程。比如：
kill <PID>  # 终止指定进程
killall <进程名>  # 终止所有同名进程

# 有时候僵尸进程的父进程可能已经退出，这样僵尸进程就无法被处理。可以使用kill命令来解决这个问题。比如：
kill -9 <PID>  # 强制终止僵尸进程的父进程

# 有时候我们需要关闭某个端口上运行的应用程序，可以通过查找该应用程序对应的PID，然后使用kill命令终止它
lsof -i:<端口号>  # 查找该端口对应的PID
kill <PID>  # 终止该应用程序


# 通过kill命令，可以向进程发送SIGSTOP信号，暂停它的执行，然后使用nice命令调整进程的优先级，并使用kill命令恢复进程的执行。比如：
kill -s STOP <PID>  # 暂停进程的执行
nice -n <优先级> <命令>  # 调整进程的优先级
kill -s CONT <PID>  # 恢复进程的执行


```