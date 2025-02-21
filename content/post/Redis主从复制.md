---
title: "Redis主从复制"
date: 2024-03-20T20:46:01+08:00
draft: false
categories: ["Redis"]
tags: ["Redis"]
keywords: ["Redis","Redis主从复制"]
---


# Redis主从复制

## 一、什么是Redis主从复制
### 1. 从复制的架构：

   Redis Replication是一种 master-slave 模式的复制机制，这种机制使得 slave 节点可以成为与 master 节点完全相同的副本，可以采用一主多从或者级联结构。

   主从复制的配置要点：
   * 配从库不配主，从库配置：slaveof 主库IP 主库端口
   * 查看redis的配置信息：info replication
### 2. Redis为什么需要主从复制？

使用Redis主从复制的原因主要是单台Redis节点存在以下的局限性：

 1.  Redis虽然读写的速度都很快，单节点的Redis能够支撑QPS大概在5w左右，如果上千万的用户访问，Redis就承载不了，成为了高并发的瓶颈。
 2.  单节点的Redis不能保证高可用，当Redis因为某些原因意外宕机时，会导致缓存不可用
 3.  CPU的利用率上，单台Redis实例只能利用单个核心，这单个核心在面临海量数据的存取和管理工作时压力会非常大。


### 3. 主从复制的好处：
1. 数据冗余：主从复制实现了数据的热备份，是持久化之外的一种数据冗余方式。
2. 故障恢复：如果master宕掉了，使用哨兵模式，可以提升一个 slave 作为新的 master，进而实现故障转移，实现高可用
3. 负载均衡：可以轻易地实现横向扩展，实现读写分离，一个 master 用于写，多个 slave 用于分摊读的压力，从而实现高并发；

### 4. 主从复制的缺点：
由于所有的写操作都是先在Master上操作，然后同步更新到Slave上，所以从Master同步到Slave服务器有一定的延迟，当系统很繁忙的时候，延迟问题会更加严重，Slave机器数量的增加也会使这个问题更加严重

## 二、主从复制的原理
从总体上来说，Redis主从复制的策略就是：当主从服务器刚建立连接的时候，进行全量同步；全量复制结束后，进行增量复制。当然，如果有需要，slave 在任何时候都可以发起全量同步。

### 1、主从全量复制的流程：
Redis全量复制一般发生在Slave初始化阶段，这时Slave需要将Master上的所有数据都复制一份，具体步骤如下：
1. slave服务器连接到master服务器，便开始进行数据同步，发送psync命令（Redis2.8之前是sync命令）
2. master服务器收到psync命令之后，开始执行bgsave命令生成RDB快照文件并使用缓存区记录此后执行的所有写命令
   * 如果master收到了多个slave并发连接请求，它只会进行一次持久化，而不是每个连接都执行一次，然后再把这一份持久化的数据发送给多个并发连接的slave。
   * 如果RDB复制时间超过60秒（repl-timeout），那么slave服务器就会认为复制失败，可以适当调节大这个参数 
3. master服务器bgsave执行完之后，就会向所有Slava服务器发送快照文件，并在发送期间继续在缓冲区内记录被执行的写命令
   client-output-buffer-limit slave 256MB 64MB 60，如果在复制期间，内存缓冲区持续消耗超过64MB，或者一次性超过256MB，那么停止复制，复制失败
4. slave服务器收到RDB快照文件后，会将接收到的数据写入磁盘，然后清空所有旧数据，在从本地磁盘载入收到的快照到内存中，同时基于旧的数据版本对外提供服务。
5. master服务器发送完RDB快照文件之后，便开始向slave服务器发送缓冲区中的写命令
6. slave服务器完成对快照的载入，开始接收命令请求，并执行来自主服务器缓冲区的写命令；
7. 如果slave node开启了AOF，那么会立即执行BGREWRITEAOF，重写AOF

### 2、增量复制：
Redis的增量复制是指在初始化的全量复制并开始正常工作之后，master服务器将发生的写操作同步到slave服务器的过程，增量复制的过程主要是master服务器每执行一个写命令就会向slave服务器发送相同的写命令，slave服务器接收并执行收到的写命令。

### 3、断点续传：
1. 什么是断点续传：
   
   当master-slave网络连接断掉后，slave重新连接master时，会触发全量复制，但是从2.8版本开始，slave与master能够在网络连接断开重连后，只从中断处继续进行复制，而不必重新同步，这就是所谓的断点续传。

        断电续传这个新特性使用psync命令，旧的实现中使用sync命令。Redis2.8版本可以检测出它所连接的服务器是否支持PSYNC命令，不支持的话使用SYNC命令。master服务器收到slave发送的psync命令后，会根据自身的情况做出对应的处理，可能是FULLRESYNC runid offset触发全量复制，也可能是CONTINUE触发增量复制
        命令格式：psync runid offset
    
2. 工作原理：
   1. master服务器在内存缓冲区中给每个slave服务器都维护了一份同步备份日志（in-memory backlog），缓存最近一段时间的数据，默认大小1m，如果超过这个大小就会清理掉。
   2. 同时，master 和 slave 服务器都维护了一个复制偏移量（replication offset）和 master线程ID（master run id），每个slave服务器在跟master服务器进行同步时都会携带master run id 和 最后一次同步的复制偏移量offset，通过offset可以知道主从之间的数据不一致的情况。
   3. 当连接断开时，slave服务器会重新连接上master服务器，然后请求继续复制。假如主从服务器的两个master run id相同，并且指定的偏移量offset在同步备份日志中还有效，复制就会从上次中断的点开始继续。如果其中一个条件不满足，就会进行完全重新同步，因为主运行id不保存在磁盘中，如果从服务器重启的话就只能进行完全同步了。master服务器维护的offset是存储在backlog中，msater就是根据slave发送的offset来从backlog中获取数据的
   4. 在部分同步过程中，master会将本地记录的同步备份日志中记录的指令依次发送给slave服务器从而达到数据一致。

### 4、无磁盘化复制：
在前面全量复制的过程中，master会将数据保存在磁盘的rdb文件中然后发送给slave服务器，但如果master上的磁盘空间有限或者是使用比较低速的磁盘，这种操作会给master服务器带来较大的压力，那怎么办呢？在Redis2.8之后，可以通过无盘复制来达到目的，由master直接开启一个socket，在内存中创建RDB文件，再将rdb文件发送给slave服务器，不使用磁盘作为中间存储。（无盘复制一般应用在磁盘空间有限但是网络状态良好的情况下）

    repl-diskless-sync ：是否开启无磁盘复制
    repl-diskless-sync-delay：等待一定时长再开始复制，因为要等更多slave重新连接过来

##  三、Redis 主从复制的配置步骤

### 步骤 1：安装 Redis

首先，你需要在主节点和从节点上安装 Redis。

假设你已经安装了 Redis，可以通过以下命令检查 Redis 是否安装成功：
``` bash
redis-server --version
```

### 步骤 2：配置主节点（Master）
1. 找到 Redis 配置文件 redis.conf，在主节点上不需要做特别的配置，默认情况下 Redis 会充当主节点。
2. 启动 Redis 实例：
   ```
   redis-server /path/to/redis.conf
   ```
主节点的配置通常不需要做任何改动，除非你有特定需求。

### 步骤 3：配置从节点（Slave）
1. 找到 Redis 配置文件 redis.conf，将从节点配置成复制模式。
2. 在从节点的 redis.conf 中添加以下配置项：
   ``` bash
   slaveof <master-ip> <master-port>
   ```
   例如，如果主节点 IP 是 192.168.1.100，端口是 6379，则配置如下：
   ```
   slaveof 192.168.1.100 6379
   ```
3. 启动 Redis 实例作为从节点：
   ``` bash
   redis-server /path/to/slave-redis.conf
   ```
   如果从节点的配置文件中已经设置了 slaveof，则它会自动连接到主节点并开始同步数据。

### 步骤 4：验证主从复制是否生效
在主节点上，可以执行以下命令来查看从节点信息：
``` bash
redis-cli -h <master-ip> -p 6379 info replication
```
这会返回主节点的复制信息，类似于：
``` bash
role:master
connected_slaves:1
slave0:ip=192.168.1.101,port=6379,state=online,offset=12345,lag=1
```
connected_slaves 表示当前连接的从节点数，slave0 表示从节点的 IP 地址、端口、状态、同步偏移量等信息。

在从节点上，你可以通过以下命令查看其状态：
``` bash
redis-cli -h <slave-ip> -p 6379 info replication
```
输出类似于：
``` bash
role:slave
master_host:192.168.1.100
master_port:6379
master_link_status:up
master_last_io_seconds_ago:5
slave_repl_offset:12345
```
这表示从节点正在成功地从主节点同步数据。

### 测试主从复制
1. 写入数据到主节点： 在主节点上使用 SET 命令写入数据：
   ``` bash
   redis-cli -h 192.168.1.100 -p 6379 SET mykey "Hello, Redis!"
   ```
2. 读取数据从从节点： 你可以从从节点读取数据，验证主从同步：
``` bash
redis-cli -h 192.168.1.101 -p 6379 GET mykey
```
如果主从复制正常工作，应该可以看到从节点返回的值为 Hello, Redis!。

#### 执行读写分离操作
在 PHP 中，你可以基于当前操作来选择使用主节点（写操作）或者从节点（读操作）。例如：
``` php
// 写操作，使用主节点
$redis = new Redis();
$redis->connect('192.168.1.100', 6379);  // 主节点 IP 和端口
$redis->set('name', 'John Doe');

// 读操作，使用从节点
$redis = new Redis();
$redis->connect('192.168.1.101', 6379);  // 从节点 IP 和端口
$value = $redis->get('name');
echo $value;  // 应该输出 "John Doe"

```

## 四、主从复制的其他问题
1. 主从复制的特点：
   1. Redis使用异步复制，每次接收到写命令之后，先在内部写入数据，然后异步发送给slave服务器。但从Redis 2.8开始，从服务器会周期性的应答从复制流中处理的数据量。
   2. Redis主从复制不阻塞master服务器。也就是说当若干个从服务器在进行初始同步时，主服务器仍然可以处理外界请求。
   3. 主从复制不阻塞slave服务器。当master服务器进行初始同步时，slave服务器返回的是以前旧版本的数据，如果你不想这样，那么在启动redis配置文件中进行设置，那么从redis在同步过程中来自外界的查询请求都会返回错误给客户端；虽然说主从复制过程中对于从redis是非阻塞的，它会用旧的数据集来提供服务，但是当初始同步完成后，需删除旧数据集和加载新的数据集，在这个短暂时间内，从服务器会阻塞连接进来的请求，对于大数据集，加载到内存的时间也是比较多的
   4. 主从复制提高了redis服务的扩展性，避免单个redis服务器的读写访问压力过大的问题，同时也可以给为数据备份及冗余提供一种解决方案；
   5. 使用主从复制可以为master服务器免除把数据写入磁盘的消耗，可以配置让master服务器不再将数据持久化到磁盘，而是通过连接让一个配置的slave类型的Redis服务器及时将相关数据持久化到磁盘。不过这种做法存在master类型的Redis服务器一旦重启，因为此时master服务器不进行持久化，所以数据为空，这时候通过主从同步可能导致slave类型的Redis服务器上的数据也被清空，所以这个配置要确保主服务器不会自动重启（详见第2点的“master开启持久化对主从架构的安全意义”）

2. master开启持久化对主从架构的安全意义：
   
   使用主从架构，必须开启master服务器的持久化机制，不建议用slave服务器作为master服务器的数据热备。当不能这么做时，比如考虑到延迟的问题，应该将master服务器配置为避免自动重启。否则，在关闭master服务器持久化机制并开始自动重启的情况下，会造成主从服务器数据被清空的情况。也就是master的持久化关闭，可能在master宕机重启的时候数据是空的（RDB和AOF都关闭了），此时就会将空数据复制到slave ，导致slave服务器的数据也丢了。

        为了更好的理解这个问题，看下面这个失败的例子，其中主服务器和从服务器中数据库都被删除了：设置节点A为主服务器，关闭持久化，节点B和C从节点A复制数据。这时出现了一个崩溃，但Redis具有自动重启系统，重启了进程，因为关闭了持久化，节点重启后只有一个空的数据集。节点B和C从节点A进行复制，现在节点A是空的，所以节点B和C上的复制数据也会被删除。当在高可用系统中使用Redis Sentinel，关闭了主服务器的持久化，并且允许自动重启，这种情况是很危险的。比如主服务器可能在很短的时间就完成了重启，以至于Sentinel都无法检测到这次失败，那么上面说的这种失败的情况就发生了。

3. 过期key的处理：
   
   对于slave服务器上过期键的处理，主要是有master服务器负责。如果master过期了一个key，则由master服务器负责键的过期删除处理，然后将相关删除命令以数据同步的方式同步给slave服务器，slave服务器根据删除命令删除本地的key。

## 五、Redis的高可用:
前面说过，通过主从复制，如果master服务器宕机了，可以提升一个slave服务器作为新的master服务器，实现故障的转移，实现高可用。也就是说，当master宕掉之后，可以手动执行“SLAVEOF no one”命令，重新选择一台服务器作为master服务器。但是呢，我们总不能保证每次master宕掉之后，都可以及时察觉并手动执行该命令，这时就可以使用“哨兵模式sentinel”，哨兵模式其实就是“SLAVEOF no one”命令的自动版，能够后台监控master是否故障，如果故障了，则根据投票数自动将slave转换为master，如果之前的master重启回来，不会造成双master冲突，因为原本的master会变成slave。

    配置步骤：
    1. 在自定义的/myredis目录下新建sentinel.conf文件（名字绝不能错）。
    2. 配置哨兵，在配置文件中写： sentinel monitor 被监控数据库名字 127.0.0.1 6379 1 最后的数字1，表示主机挂掉后salve投票看让谁接替成为主机，得票数多少后成为主机。
    3. 启动哨兵：redis-sentinel /myredis/sentinel.conf
   