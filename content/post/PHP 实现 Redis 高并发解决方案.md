---
title: "PHP 实现 Redis 高并发解决方案"
date: 2025-03-28T22:33:45+08:00
draft: false
categories: ["Redis"]
tags: ["Redis"]
keywords: ["Redis","高并发"]
---



## PHP 实现 Redis 高并发解决方案

使用 PHP 实现 Redis 在缓存加速、分布式锁和队列场景中的应用。

首先确保已安装 PHP Redis 扩展

## 一、缓存加速实现
### 1. 基本缓存操作
```php
<?php
$redis = new Redis();
$redis->connect('127.0.0.1', 6379);

// 设置缓存
function setCache($key, $value, $expire = 3600) {
    global $redis;
    $serialized = serialize($value);
    return $redis->setex($key, $expire, $serialized);
}

// 获取缓存
function getCache($key) {
    global $redis;
    $serialized = $redis->get($key);
    return $serialized ? unserialize($serialized) : false;
}

// 删除缓存
function deleteCache($key) {
    global $redis;
    return $redis->del($key);
}

// 示例：用户数据缓存
function getUser($userId) {
    $cacheKey = "user:{$userId}";
    $user = getCache($cacheKey);
    
    if ($user === false) {
        // 模拟数据库查询
        $user = [
            'id' => $userId,
            'name' => 'User ' . $userId,
            'email' => "user{$userId}@example.com"
        ];
        // 写入缓存，有效期1小时
        setCache($cacheKey, $user, 3600);
    }
    
    return $user;
}

// 使用示例
$user = getUser(123);
print_r($user);
```

### 2. 防止缓存穿透
```php
function getProduct($productId) {
    global $redis;
    $cacheKey = "product:{$productId}";
    $product = getCache($cacheKey);
    
    if ($product === false) {
        // 使用互斥锁防止缓存击穿
        $lockKey = "lock:product:{$productId}";
        $locked = $redis->set($lockKey, 1, ['nx', 'ex' => 10]);
        
        if ($locked) {
            try {
                // 模拟数据库查询
                $product = [
                    'id' => $productId,
                    'name' => 'Product ' . $productId,
                    'price' => rand(100, 1000)
                ];
                
                if (empty($product)) {
                    // 缓存空对象防止穿透，有效期5分钟
                    setCache($cacheKey, [], 300);
                } else {
                    setCache($cacheKey, $product, 3600);
                }
            } finally {
                $redis->del($lockKey);
            }
        } else {
            // 等待其他进程完成缓存设置
            usleep(500000); // 等待500ms
            return getProduct($productId); // 重试
        }
    }
    
    return $product ?: null;
}
```

## 二、 分布式锁实现
### 1. 基本分布式锁
```php
class RedisLock {
    private $redis;
    private $lockKey;
    private $identifier;
    private $lockTimeout;
    
    public function __construct($redis, $lockKey, $lockTimeout = 10) {
        $this->redis = $redis;
        $this->lockKey = $lockKey;
        $this->lockTimeout = $lockTimeout;
        $this->identifier = uniqid();
    }
    
    public function acquire($waitTimeout = 5) {
        $end = microtime(true) + $waitTimeout;
        
        while (microtime(true) < $end) {
            if ($this->redis->set(
                $this->lockKey, 
                $this->identifier, 
                ['nx', 'ex' => $this->lockTimeout]
            )) {
                return true;
            }
            
            usleep(10000); // 等待10ms
        }
        
        return false;
    }
    
    public function release() {
        $script = '
            if redis.call("get", KEYS[1]) == ARGV[1] then
                return redis.call("del", KEYS[1])
            else
                return 0
            end
        ';
        
        return $this->redis->eval($script, [$this->lockKey, $this->identifier], 1);
    }
    
    public function __destruct() {
        $this->release();
    }
}

// 使用示例
$lock = new RedisLock($redis, 'lock:order:123');
if ($lock->acquire()) {
    try {
        // 执行需要加锁的操作
        echo "Lock acquired, doing critical section...\n";
        sleep(2);
    } finally {
        $lock->release();
    }
} else {
    echo "Failed to acquire lock\n";
}
```
###  2. 可重入锁实现
```php
class ReentrantRedisLock extends RedisLock {
    private $heldCount = 0;
    
    public function acquire($waitTimeout = 5) {
        // 检查是否已经持有锁
        if ($this->heldCount > 0) {
            $this->heldCount++;
            return true;
        }
        
        $acquired = parent::acquire($waitTimeout);
        if ($acquired) {
            $this->heldCount = 1;
        }
        return $acquired;
    }
    
    public function release() {
        if ($this->heldCount > 1) {
            $this->heldCount--;
            return true;
        }
        
        $released = parent::release();
        if ($released) {
            $this->heldCount = 0;
        }
        return $released;
    }
}
```

## 三、队列实现
### 1. 简单队列
```php
// 生产者
function enqueue($queueName, $data) {
    global $redis;
    return $redis->lPush($queueName, json_encode($data));
}

// 消费者
function dequeue($queueName, $timeout = 30) {
    global $redis;
    $result = $redis->brPop($queueName, $timeout);
    return $result ? json_decode($result[1], true) : null;
}

// 示例
enqueue('email_queue', [
    'to' => 'user@example.com',
    'subject' => 'Welcome',
    'body' => 'Thank you for registering'
]);

$task = dequeue('email_queue');
if ($task) {
    // 处理任务
    echo "Sending email to: {$task['to']}\n";
}
```
### 2. 延迟队列
```php
function enqueueDelayed($queueName, $data, $delaySeconds) {
    global $redis;
    $score = time() + $delaySeconds;
    return $redis->zAdd($queueName, $score, json_encode($data));
}

function processDelayedQueue($queueName) {
    global $redis;
    $now = time();
    $items = $redis->zRangeByScore($queueName, 0, $now);
    
    if (!empty($items)) {
        // 使用事务移除已处理项
        $redis->multi();
        foreach ($items as $item) {
            $redis->zRem($queueName, $item);
        }
        $redis->exec();
        
        return array_map(function($item) {
            return json_decode($item, true);
        }, $items);
    }
    
    return [];
}

// 示例
enqueueDelayed('reminder_queue', ['user_id' => 123, 'message' => 'Pay your bill'], 60);

$tasks = processDelayedQueue('reminder_queue');
foreach ($tasks as $task) {
    echo "Sending reminder to user {$task['user_id']}: {$task['message']}\n";
}
```
### 3. Pub/Sub 模式
```php
// 发布者
function publishMessage($channel, $message) {
    global $redis;
    return $redis->publish($channel, json_encode($message));
}

// 订阅者
function subscribe($channels, $callback) {
    global $redis;
    $pubsub = $redis->pSubscribe($channels);
    
    try {
        foreach ($pubsub as $message) {
            if ($message->kind === 'message') {
                $data = json_decode($message->payload, true);
                call_user_func($callback, $message->channel, $data);
            }
        }
    } catch (Exception $e) {
        $redis->pUnsubscribe();
        throw $e;
    }
}

// 示例使用
// 在一个进程中
publishMessage('notifications', ['user_id' => 123, 'text' => 'New message']);

// 在另一个进程中
subscribe(['notifications'], function($channel, $message) {
    echo "Received on {$channel}: ";
    print_r($message);
});
```

## 四、综合应用示例 - 秒杀系统
``` php
class SeckillService {
    private $redis;
    private $productKey;
    
    public function __construct($redis, $productId) {
        $this->redis = $redis;
        $this->productKey = "seckill:product:{$productId}";
    }
    
    public function initInventory($inventory) {
        // 设置商品库存
        $this->redis->set($this->productKey, $inventory);
    }
    
    public function seckill($userId) {
        // 使用分布式锁
        $lock = new RedisLock($this->redis, "lock:{$this->productKey}");
        
        if (!$lock->acquire(1)) {
            return ['success' => false, 'message' => '系统繁忙'];
        }
        
        try {
            // 检查库存
            $inventory = $this->redis->get($this->productKey);
            if ($inventory <= 0) {
                return ['success' => false, 'message' => '已售罄'];
            }
            
            // 扣减库存
            $this->redis->decr($this->productKey);
            
            // 记录订单 (实际应用中应该入队列异步处理)
            $orderId = uniqid();
            $this->redis->hSet("seckill:orders", $orderId, json_encode([
                'user_id' => $userId,
                'product_key' => $this->productKey,
                'created_at' => time()
            ]));
            
            return ['success' => true, 'order_id' => $orderId];
        } finally {
            $lock->release();
        }
    }
}

// 使用示例
$redis = new Redis();
$redis->connect('127.0.0.1', 6379);

$seckill = new SeckillService($redis, 1001);
$seckill->initInventory(100); // 初始化100件库存

// 模拟并发请求
for ($i = 0; $i < 5; $i++) {
    $result = $seckill->seckill(rand(1000, 9999));
    print_r($result);
}
```

## 五、最佳实践建议
### 1. 连接管理：使用连接池或持久连接
```php
$redis = new Redis();
$redis->pconnect('127.0.0.1', 6379);
```
### 2. 错误处理：添加重试机制
```php
function redisRetry($callback, $maxRetries = 3) {
    $retries = 0;
    while ($retries < $maxRetries) {
        try {
            return $callback();
        } catch (RedisException $e) {
            $retries++;
            if ($retries >= $maxRetries) {
                throw $e;
            }
            usleep(100000 * $retries); // 指数退避
        }
    }
}
```
### 3. 性能优化：使用管道(pipeline)
```php
$pipe = $redis->pipeline();
$pipe->set('key1', 'value1');
$pipe->set('key2', 'value2');
$pipe->incr('counter');
$pipe->expire('key1', 60);
$results = $pipe->exec();
```
### 4. Lua脚本：保证原子性
```php
$script = '
    local current = redis.call("GET", KEYS[1])
    if current == ARGV[1] then
        return redis.call("INCRBY", KEYS[1], ARGV[2])
    else
        return nil
    end
';
$result = $redis->eval($script, ['counter', 'expected_value', 'increment'], 1);
```