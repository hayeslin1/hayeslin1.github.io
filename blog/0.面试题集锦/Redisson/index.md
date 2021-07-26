# 分布式锁

分布式锁的实现有多种， redis/zk/mysql 等等都可以实现

## 1.synchronized
```java
synchronized (this) {
    if (getDB > 0) {
        getDB--;
        System.out.println("库存扣减成功：" + getDB);
    } else {
        System.out.println("库存不足");
    }
}
```

synchronized可以实现单机模式下的并发模式，但是当分布式模式的时候就不行了

synchronized只在单机生效，分布式就嗝屁了。 

## 2. redis > setnx

### 格式： setnx key value 

SETNX：是「 set if not exists 」 如果不存在则set

将key设置value， 仅当key不存在的时候才会操作成功。 

若给定的key已经存在， 不做任何操作。

```java

//加锁
String key = "lock";
Boolean ifAbsent = redisTemplate.opsForValue().setIfAbsent(key, "1", 1, TimeUnit.MINUTES);
if (!ifAbsent){
    System.out.println("正在进行中。。。。");
}
try {
    if (getDB > 0) {
        getDB--;
        System.out.println("库存扣减成功：" + getDB);
    } else {
        System.out.println("库存不足");
    }
} finally {
    //用完删除
    redisTemplate.delete(key);
}

```

问题分析： 

1. 当执行到扣减库存的代码， 被强制kill -9 , 此时需要等待10分钟才会自动删除key， 对于电商秒杀，直接杀程序员几天， 损失巨大。
2. 可以将时间缩短， 比如10s。 --》 问题：如果存在慢查询sql，设置的10s不够用，锁被自动释放，同样其他线程可以进入，高并发下同样不起作用。
3. thread-1没执行完，锁被释放，thread-2进入，thread-1执行删除key， thread-3进入，thread-2执行到删除key， thread-4进入。 由此可见程序全乱了。 
4. 可以给key设置一个随机ID， 删除的时候的查看一下是不是当前线程 。但是时间还是不合适，设置长了对于秒杀不行， 设置短了对于慢查询不行。 
5. 解决方案，增加一个子进程， 来监控是否执行完毕， 未执行完毕，时间到期可以自动进行续期
6. 思路很简单， 自己实现上述代码非常复杂。
   

redisson转为解决这类问题的。


## 3. redisson

转为分布式的而生 

```java 
//底层实现的加锁代码： （LUA脚本）
<T> RFuture<T> tryLockInnerAsync(long waitTime, long leaseTime, TimeUnit unit, long threadId, RedisStrictCommand<T> command) {
    return evalWriteAsync(getRawName(), LongCodec.INSTANCE, command,
        "if (redis.call('exists', KEYS[1]) == 0) then " +
                "redis.call('hincrby', KEYS[1], ARGV[2], 1); " +
                "redis.call('pexpire', KEYS[1], ARGV[1]); " +
                "return nil; " +
                "end; " +
                "if (redis.call('hexists', KEYS[1], ARGV[2]) == 1) then " +
                "redis.call('hincrby', KEYS[1], ARGV[2], 1); " +
                "redis.call('pexpire', KEYS[1], ARGV[1]); " +
                "return nil; " +
                "end; " +
                "return redis.call('pttl', KEYS[1]);",
        Collections.singletonList(getRawName()), unit.toMillis(leaseTime), getLockName(threadId));
}


// 子进程 续期源码

private void renewExpiration() {
    ExpirationEntry ee = EXPIRATION_RENEWAL_MAP.get(getEntryName());
    if (ee == null) {
        return;
    }
    
    Timeout task = commandExecutor.getConnectionManager().newTimeout(new TimerTask() {
        @Override
        public void run(Timeout timeout) throws Exception {
            ExpirationEntry ent = EXPIRATION_RENEWAL_MAP.get(getEntryName());
            if (ent == null) {
                return;
            }
            Long threadId = ent.getFirstThreadId();
            if (threadId == null) {
                return;
            }
            
            RFuture<Boolean> future = renewExpirationAsync(threadId);
            future.onComplete((res, e) -> {
                if (e != null) {
                    log.error("Can't update lock " + getRawName() + " expiration", e);
                    EXPIRATION_RENEWAL_MAP.remove(getEntryName());
                    return;
                }
                
                if (res) {
                    // reschedule itself
                    renewExpiration();
                }
            });
        }
    }, internalLockLeaseTime / 3, TimeUnit.MILLISECONDS);
    
    ee.setTimeout(task);
}

// internalLockLeaseTime 默认值 30 * 1000  ms
// 每10s 执行一次


```

CAP理论： 

C：一致性

A：可用性

P：容错性


大部分的高并发高可用都是AP， 高可用场景允许有微小的错误 ， 

zk是CP， 不出错
