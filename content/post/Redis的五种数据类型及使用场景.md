---
title: "Redis的五种数据类型及使用场景"
date: 2023-09-01T20:46:01+08:00
draft: false
categories: ["Redis"]
tags: ["Redis"]
keywords: ["Redis"]
---


# Redis的五种数据类型及使用场景

## 一、字符串（String）
* string 是 redis 最基本的类型，你可以理解成与 Memcached 一模一样的类型，一个 key 对应一个 value。
* string 类型是二进制安全的。意思是 redis 的 string 可以包含任何数据。比如 jpg 图片或者序列化的对象。
* string 类型是 Redis 最基本的数据类型，string 类型的值最大能存储 512MB。

### 常用命令：
* SET key value  设置指定 key 的值。
* GET key        获取指定 key 的值。

### 应用场景：
String 是最常用的一种数据类型，普通的 key/value 存储都可以归为此类，即可以完全实现目前 Memcached 的功能，并且效率更高。还可以享受 Redis 的定时持久化，操作日志及 Replication 等功能。除了提供与 Memcached 一样的 get、set、incr、decr 等操作外，Redis 还提供了下面一些操作：
- 获取字符串长度
- 往字符串 append 内容
- 设置和获取字符串的某一段内容
- 设置及获取字符串的某一位（bit）
- 批量设置一系列字符串的内容
  
### 使用场景：
常规 key-value 缓存应用。常规计数：微博数，粉丝数。

### 实现方式：
String 在 redis 内部存储默认就是一个字符串，被 redisObject 所引用，当遇到 incr,decr 等操作时会转成数值型进行计算，此时 redisObject 的 encoding 字段为 int


## 二、 哈希（Hash）
Redis hash 是一个键值 (key=>value) 对集合。

Redis hash 是一个 string 类型的 field 和 value 的映射表，hash 特别适合用于存储对象

Redis 的 Hash 实际是内部存储的 Value 为一个 HashMap，并提供了直接存取这个 Map 成员的接口

注意，Redis 提供了接口 (hgetall) 可以直接取到全部的属性数据，但是如果内部 Map 的成员很多，那么涉及到遍历整个内部 Map 的操作，由于 Redis 单线程模型的缘故，这个遍历操作可能会比较耗时，而另其它客户端的请求完全不响应，这点需要格外注意。

### 常用命令：
* HSET key field value    // Redis Hset 命令用于为哈希表中的字段赋值 。如果哈希表不存在，一个新的哈希表被创建并进行 HSET 操作。 如果字段已经存在于哈希表中，旧值将被覆盖。
* HMSET key field1 value1 [field2 value2 ]  // 同时将多个 field-value (域-值)对设置到哈希表 key 中。
*	HGET key field          // 获取存储在哈希表中指定字段的值。
* HGETALL key             // 获取在哈希表中指定 key 的所有字段和值
* HEXISTS key field           // 查看哈希表 key 中，指定的字段是否存在。
* HKEYS key               // 获取哈希表中的所有字段
* HLEN key                // 获取哈希表中字段的数量
* HDEL key field1 [field2] //删除一个或多个哈希表字段
hget,hset,hgetall 等。

### 使用场景：
存储部分变更数据，如用户信息等。

### 实现方式：
Redis Hash 对应 Value 内部实际就是一个 HashMap，实际这里会有 2 种不同实现，
这个 Hash 的成员比较少时 Redis 为了节省内存会采用类似一维数组的方式来紧凑存储，而不会采用真正的 HashMap 结构，对应的 value redisObject 的 encoding 为 zipmap，当成员数量增大时会自动转成真正的 HashMap，此时 encoding 为 ht


## 三、列表（List）
列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部（左边）或者尾部（右边）
### 常用命令：
* LPUSH key value1 [value2]     // 将一个或多个值插入到列表头部
* LPUSHX key value              // 将一个值插入到已存在的列表头部
* RPUSH key value1 [value2]     // 在列表中添加一个或多个值到列表尾部
* RPUSHX key value              // 为已存在的列表添加值
* LPOP key                      // 移出并获取列表的第一个元素
* RPOP key                      // 移除列表的最后一个元素，返回值为移除的元素。
* LRANGE key start stop         // 获取列表指定范围内的元素
* LLEN key                      // 获取列表长度
* LINDEX key index              // 通过索引获取列表中的元素
lpush（添加左边元素）
rpush,
lpop（移除左边第一个元素）
rpop,
lrange（获取列表片段，LRANGE key start stop）等

### 应用场景：

Redis list 的应用场景非常多，也是 Redis 最重要的数据结构之一，比如 twitter 的关注列表，粉丝列表等都可以用 Redis 的 list 结构来实现。

List 就是链表，相信略有数据结构知识的人都应该能理解其结构。使用 List 结构，我们可以轻松地实现最新消息排行等功能。List 的另一个应用就是消息队列，

可以利用 List 的 PUSH 操作，将任务存在 List 中，然后工作线程再用 POP 操作将任务取出进行执行。Redis 还提供了操作 List 中某一段的 api，你可以直接查询，删除 List 中某一段的元素。

### 实现方式：
Redis list 的实现为一个双向链表，即可以支持反向查找和遍历，更方便操作，不过带来了部分额外的内存开销，Redis 内部的很多实现，包括发送缓冲队列等也都是用的这个数据结构。

Redis 的 list 是每个子元素都是 String 类型的双向链表，可以通过 push 和 pop 操作从列表的头部或者尾部添加或者删除元素，这样 List 即可以作为栈，也可以作为队列。 获取越接近两端的元素速度越快，但通过索引访问时会比较慢。

### 使用场景
 消息队列系统：使用 list 可以构建队列系统，使用 sorted set 甚至可以构建有优先级的队列系统。比如：将 Redis 用作日志收集器，实际上还是一个队列，多个端点将日志信息写入 Redis，然后一个 worker 统一将所有日志写到磁盘。

 取最新 N 个数据的操作：记录前 N 个最新登陆的用户 Id 列表，超出的范围可以从数据库中获得


 ## 四、集合（Set）
 Redis 的 Set 是 String 类型的无序集合。  
 集合成员是唯一的，这就意味着集合中不能出现重复的数据。   
 集合对象的编码可以是 intset 或者 hashtable。 Redis 中集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是 O(1)。  
 集合中最大的成员数为 232 - 1 (4294967295, 每个集合可存储40多亿个成员)。

### 常用命令：
* SADD key member1 [member2]          // 向集合添加一个或多个成员
* SCARD key                           // 获取集合的成员数
* SMEMBERS key                        // 返回集合中的所有成员
* SUNION key1 [key2]                  // 返回所有给定集合的并集
* SPOP key                            // 移除并返回集合中的一个随机元素
sadd,spop,smembers,sunion 等。

### 应用场景：
Redis set 对外提供的功能与 list 类似是一个列表的功能，特殊之处在于 set 是可以自动排重的，当你需要存储一个列表数据，又不希望出现重复数据时，set 是一个很好的选择，并且 set 提供了判断某个成员是否在一个 set 集合内的重要接口，这个也是 list 所不能提供的。

Set 就是一个集合，集合的概念就是一堆不重复值的组合。利用 Redis 提供的 Set 数据结构，可以存储一些集合性的数据。

案例：在微博中，可以将一个用户所有的关注人存在一个集合中，将其所有粉丝存在一个集合。Redis 还为集合提供了求交集、并集、差集等操作，可以非常方便的实现如共同关注、共同喜好、二度好友等功能，对上面的所有集合操作，你还可以使用不同的命令选择将结果返回给客户端还是存集到一个新的集合中。

### 实现方式：
set 的内部实现是一个 value 永远为 null 的 HashMap，实际就是通过计算 hash 的方式来快速排重的，这也是 set 能提供判断一个成员是否在集合内的原因

## 五、sorted set（有序集合）
Redis 有序集合和集合一样也是 string 类型元素的集合,且不允许重复的成员。  
不同的是每个元素都会关联一个 double 类型的分数。redis 正是通过分数来为集合中的成员进行从小到大的排序。  
有序集合的成员是唯一的,但分数(score)却可以重复。
集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是 O(1)。 集合中最大的成员数为 232 - 1 (4294967295, 每个集合可存储40多亿个成员)。

### 常用命令：
* ZADD key score1 member1 [score2 member2]          // 向有序集合添加一个或多个成员，或者更新已存在成员的分数
* ZCARD key                                         // 获取有序集合的成员数
* ZRANGE key start stop [WITHSCORES]                // 通过索引区间返回有序集合指定区间内的成员
* ZREM key member [member ...]                      // 移除有序集合中的一个或多个成员
zadd,zrange,zrem,zcard 等

### 使用场景：
Redis sorted set 的使用场景与 set 类似，区别是 set 不是自动有序的，而 sorted set 可以通过用户额外提供一个优先级 (score) 的参数来为成员排序，并且是插入有序的，即自动排序。当你需要一个有序的并且不重复的集合列表，那么可以选择 sorted set 数据结构，比如 twitter 的 public timeline 可以以发表时间作为 score 来存储，这样获取时就是自动按时间排好序的。和 Set 相比，Sorted Set 关联了一个 double 类型权重参数 score，使得集合中的元素能够按 score 进行有序排列，redis 正是通过分数来为集合中的成员进行从小到大的排序。zset 的成员是唯一的，但分数 (score) 却可以重复。比如一个存储全班同学成绩的 Sorted Set，其集合 value 可以是同学的学号，而 score 就可以是其考试得分，这样在数据插入集合的时候，就已经进行了天然的排序。另外还可以用 Sorted Set 来做带权重的队列，比如普通消息的 score 为 1，重要消息的 score 为 2，然后工作线程可以选择按 score 的倒序来获取工作任务。让重要的任务优先执行。


### 实现方式：
Redis sorted set 的内部使用 HashMap 和跳跃表 (SkipList) 来保证数据的存储和有序，HashMap 里放的是成员到 score 的映射，而跳跃表里存放的是所有的成员，排序依据是 HashMap 里存的 score, 使用跳跃表的结构可以获得比较高的查找效率，并且在实现上比较简单。


## PHP 操作 Redis 的基本方法
通过 Composer 安装  [predis/predis](https://packagist.org/packages/predis/predis)
   ```
   composer require predis/predis
   ```
1. redis Strng (字符串) 的简单操作
```php
      //设置一个字符串
       Redis::set('name',"hello");
       $value = Redis::get('name');
       var_dump($value);   //hello
       //重复set
       Redis::set('name','world');
       $value1 = Redis::get('name');
       var_dump($value1);   //world
```
2. redis list 的简单操作
   ```php
     public function RedisList()
    {
      //左边添加元素
      Redis::del('list');          //清空表
      Redis::lpush('list','book1');
      Redis::lpush('list','book2');
      Redis::lpush('list','book3');
      $value = Redis::lrange('list',0,-1); //获取所有值
      var_dump($value);  //["book3","book2","book1"]
      //从右侧添加元素
      Redis::rpush('list','book4');
      $value = Redis::lrange('list',0,-1); //获取所有值
      var_dump($value);  //["book3","book2","book1","book4"]
      Redis::lpop('list');
      $value = Redis::lrange('list',0,-1);  //["book2","book1","book4"]
      Redis::rpop('list');
      $value = Redis::lrange('list',0,-1);  //["book2","book1"]
      return $value;
    }
    ```
3. redis hash 的简单操作
   ```php
   public function RedisHash()
    {
      Redis::del('hash');          //清空表
      //给哈hash表中某个key设置value
      ////如果没有则设置成功,返回1,如果存在会替换原有的值,返回0,失败返回0
      Redis::hset('hash','book','book');
      Redis::hset('hash','book','book');
      Redis::hset('hash','book','book1');
      Redis::hset('hash','dog','dog');
      Redis::hset('hash','banaa','banna');
      Redis::hset('hash','apple','apple');
      //获取hash中某个key的值
      $value = Redis::hget('hash','book'); //book1
     //获取hash中的所有值，hash中所有值是随机的
      $arr = Redis::hvals('hash'); // ["book1","dog","banna","apple"]
      //获取hash中的所有key
      $arr = Redis::hkeys('hash'); //["book","dog","banaa","apple"]
      //获取 hash 中所有的key-value
      $arr = Redis::hgetall('hash');
      //{"book":"book1","dog":"dog","banaa":"banna","apple":"apple"}
      //获取 hash 中 key的数量
      $key = Redis::hlen('hash');
      //删除某个key 如果表不存在或key不存在则返回false
      $del = Redis::hdel('hash','dog');
      $arr = Redis::hgetall('hash');
      //{"book":"book1","banaa":"banna","apple":"apple"}
      return $arr;
    }
    ```
4. redis Set (集合) 的简单操作
   ```php
   public function RedisSet($value='')
    {

       Redis::del('set');          //清空表
       // 添加元素
       Redis::sadd('set', 'book');  //重复添加返回0
       Redis::sadd('set','dog');
       Redis::sadd('set','banaa');
       Redis::sadd('set','apple');
       // 查看集合中所有的元素
       $set = Redis::smembers('set');
       //删除集合中的元素
       Redis::srem('set', 'book');
       //判断元素是否在集合中
       Redis::sismember('set','apple');  //返回 true / false
       //查看集合成员的数量
       Redis::scard('set');
       //移除集合中的一个元素
       //Redis::spop('set');  //返回的是 被移除的元素

       Redis::sadd('set1','dog');
       Redis::sadd('set1','banaa');
       Redis::sadd('set1','egg');
       //返回集合的交集
       Redis::sinter('set','set1');
       //返回集合的交集 并存到新的集合中
       Redis::sinterstore('newset','set','set1');
       // 集合合并
       Redis::sunionstore('output','set','set1');
       //var_dump(Redis::smembers('newset'));
       var_dump(Redis::smembers('output'));
    }
    ```
5. redis Sorted Set (有序集合) 的简单操作
   ```php
   public function RedisZset($value='')
    {
      // code...
      Redis::del('zset');          //清空表
      // 添加元素
      Redis::zadd('zset',1, 'book');  //重复添加返回0
      Redis::zadd('zset',2,'dog');
      Redis::zadd('zset',3,'banaa');
      Redis::zadd('zset',4,'apple');

      // 查看集合中所有的元素
      $zset = Redis::zrange('zset',0,-1);
      //返回 元素的score
      $score = Redis::zscore('zset','dog');
      //删除集合中的元素
      Redis::zrem('zset', 'book');
      //返回集合中介于 min 与 max值得个数
      Redis::zcount('zset',3,5);
      //返回有序集合中score介于min和max之间的值
      $arr = Redis::zrangebyscore('zset',3,5);
      return $arr;
    }
    ```