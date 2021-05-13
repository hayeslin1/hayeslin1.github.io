# Redis 

> Redis 是一个开源（BSD许可）的，内存中的数据结构存储系统，它可以用作数据库、缓存和消息中间件。 它支持多种类型的数据结构，如 字符串（strings）， 散列（hashes）， 列表（lists）， 集合（sets）， 有序集合（sorted sets） 与范围查询， bitmaps， hyperloglogs 和 地理空间（geospatial） 索引半径查询。 Redis 内置了 复制（replication），LUA脚本（Lua scripting）， LRU驱动事件（LRU eviction），事务（transactions） 和不同级别的 磁盘持久化（persistence）， 并通过 Redis哨兵（Sentinel）和自动 分区（Cluster）提供高可用性（high availability）。

## redis和memcached有啥区别？

+ Redis有更多的数据结构和并支持更丰富的数据操作
+ 在 redis3.x 版本中，便能支持 cluster 模式，而 memcached 没有原生的集群模式，需要依靠客户端来实现往集群中分片写入数据。
+ Redis数据可以进行持久化： RDB全量缓存，AOF增量缓存


## Redis为什么会这么快

+ 1、Redis是纯内存操作，需要的时候需要我们手动持久化到硬盘中

+ 2、Redis是单线程，从而避开了多线程中上下文频繁切换的操作。

+ 3、Redis数据结构简单、对数据的操作也比较简单

+ 4、使用底层模型不同，它们之间底层实现方式以及与客户端之间通信的应用协议不一样，Redis直接自己构建了VM 机制 ，因为一般的系统调用系统函数的话，会浪费一定的时间去移动和请求

+ 5、使用多路I/O复用模型，非阻塞I/O




## 雪崩，穿透和击穿 ， 如何解决 ？ 

+ 雪崩：当某一个时刻出现大规模的缓存失效的情况，那么就会导致大量的请求直接打在数据库上面，导致数据库压力巨大，如果在高并发的情况下，可能瞬间就会导致数据库宕机。
    + 1. 在原有的失效时间上加上一个随机值，把失效时间分散开。
    + 2. 兜底：增加熔断机制
+ 穿透：直接请求了一个本身无效的key，自然缓存中没有，DB中也没有，当频繁的类似这种请求也会把DB打死
    + 1. key本身是有范围的，判断是否是有效key
    + 2. 把无效的key也存入到redis，value=null ，设置较短的过期时间
    + 3. 使用布隆过滤器（Bloom Filter）
     > 有一种概率型数据结构，它可以告诉你key一定不存在，但是不能说明一定存在，有一定的误判率。通过hash计算出一个byte的数字，放入桶中。当判断key是否存在，返回0说明一定不存在。
+ 击穿：跟缓存雪崩有点类似，缓存雪崩是大规模的key失效，而缓存击穿是一个热点的Key，有大并发集中对其进行访问，突然间这个Key失效了，导致大并发全部打在数据库上，导致数据库压力剧增。
    + 1. 如果业务允许的话，对于热点的key可以设置永不过期的key。
    + 2. 使用互斥锁。（代码如下）
    ```java 
    /**
     * 分布式锁获取数据，解决缓存击穿
     * @param key
     * @return
     * @throws InterruptedException
     */
    public Object getData(String key) throws InterruptedException {
        //从redis查询数据
        Object result = get(key);
        ReentrantLock reentrantLock = new ReentrantLock();
        if(null == result){
            //获取锁
            if(reentrantLock.tryLock()){
                result = get(key);
                if(null == result) {
                    result = db.get(key)
                    //存缓存
                    set(key,result);
                }
                //释放锁
                reentrantLock.unlock();
            }else {
                //睡一会再拿
                Thread.sleep(100L);
                result = getData(key);
            }
        }
        return result;
    }
    ```

## 持久化

+ RDB：是将数据生成快照并存储到磁盘等介质上。
    + RDB会单独fork一个子进程来使用cow（Copy On Write ）机制持久化.主进程继续对外提供服务。 
    + 即使每5分钟持久化一次，当突然断电也会丢失5分钟内的数据。
    + 优点是恢复的快

+ AOF：记录执行过的所有指令。数据完整度更高，但是恢复慢。
+ 小孩才二选一，大人两个都要。
    + 先采用RDB持久化数据，AOF记录5分钟内的指令，这样更可靠一些。



## 过期策略
+ 定期删除：默认每隔100ms随机抽取一个key检查是否过期，过期就删除。
+ 惰性删除：定期没扫到，当查询的时候检查一下是否过期，过期的返回null删除。
+ 内存淘汰机制-LRU ： 倘若上面两种方式都没有扫到，当内容不足容纳新数据的时候，删除最近最少使用的key


## 做分布式锁 

|分类|方案|实现原理|优点|缺点|
|-|-|-|-|-|
|基于数据库|基于mysql表唯一索引|1.表增加唯一索引<br>2.加锁：执行insert语句，若报错，则表明加锁失败<br>3.解锁：执行delete语句|完全利用DB现有能力，实现简单|1.锁无超时自动失效机制，有死锁风险<br>2.不支持锁重入，不支持阻塞等待<br>3.操作数据库开销大，性能不高|
|基于缓存|基于redis命令|1. 加锁：执行setnx，若成功再执行expire添加过期时间<br>2. 解锁：执行delete命令|实现简单，相比数据库和分布式系统的实现，该方案最轻，性能最好|1.setnx和expire分2步执行，非原子操作；若setnx执行成功，但expire执行失败，就可能出现死锁<br>2.delete命令存在误删除非当前线程持有的锁的可能<br>3.不支持阻塞等待、不可重入|
|基于缓存|基于redis Lua脚本能力|1. 加锁：执行SET lock_name random_value EX seconds NX 命令<br>2. 解锁：执行Lua脚本，释放锁时验证random_value <br>-- ARGV[1]为random_value,  KEYS[1]为lock_name<br>if redis.call("get", KEYS[1]) == ARGV[1] then<br>return redis.call("del",KEYS[1])<br>else<br>return 0<br>end|同上；实现逻辑上也更严谨，除了单点问题，生产环境采用用这种方案，问题也不大。|不支持锁重入，不支持阻塞等待|


## redisson 

redisson保持了简单易用、支持锁重入、支持阻塞等待、Lua脚本原子操作

### 1、加锁Lua脚本

|参数|示例值|含义|
|-|-|-|
|KEY个数|	1|	KEY个数|
|KEYS[1]|	my_first_lock_name|	锁名|
|ARGV[1]|	60000|	持有锁的有效时间：毫秒|
|ARGV[2]|	58c62432-bb74-4d14-8a00-9908cc8b828f:1|	唯一标识：获取锁时set的唯一值，实现上为redisson客户端|ID(UUID)+线程ID|

```java
// 若锁不存在：则新增锁，并设置锁重入计数为1、设置锁过期时间
if (redis.call('exists', KEYS[1]) == 0) then
    redis.call('hset', KEYS[1], ARGV[2], 1);
    redis.call('pexpire', KEYS[1], ARGV[1]);
    return nil;
end;
 
// 若锁存在，且唯一标识也匹配：则表明当前加锁请求为锁重入请求，故锁重入计数+1，并再次设置锁过期时间
if (redis.call('hexists', KEYS[1], ARGV[2]) == 1) then
    redis.call('hincrby', KEYS[1], ARGV[2], 1);
    redis.call('pexpire', KEYS[1], ARGV[1]);
    return nil;
end;
 
// 若锁存在，但唯一标识不匹配：表明锁是被其他线程占用，当前线程无权解他人的锁，直接返回锁剩余过期时间
// 当且仅当返回nil，才表示加锁成功；客户端需要感知加锁是否成功的结果
return redis.call('pttl', KEYS[1]);
```

### 2、解锁Lua脚本

|参数|	示例值|	含义|
|-|-|-|
|KEY个数|	2|	KEY个数|
|KEYS[1]|	my_first_lock_name|	锁名|
|KEYS[2]|	redisson_lock__channel:{my_first_lock_name}	|解锁消息PubSub频道|
|ARGV[1]|	0|	redisson定义0表示解锁消息|
|ARGV[2]|	30000|	设置锁的过期时间；默认值30秒|
|ARGV[3]|	58c62432-bb74-4d14-8a00-9908cc8b828f:1|	唯一标识；同加锁流程|

```java 

// 若锁不存在：则直接广播解锁消息，并返回1
if (redis.call('exists', KEYS[1]) == 0) then
    redis.call('publish', KEYS[2], ARGV[1]);
    return 1; 
end;
 
// 若锁存在，但唯一标识不匹配：则表明锁被其他线程占用，当前线程不允许解锁其他线程持有的锁
if (redis.call('hexists', KEYS[1], ARGV[3]) == 0) then
    return nil;
end; 
 
// 若锁存在，且唯一标识匹配：则先将锁重入计数减1
local counter = redis.call('hincrby', KEYS[1], ARGV[3], -1); 
if (counter > 0) then 
    // 锁重入计数减1后还大于0：表明当前线程持有的锁还有重入，不能进行锁删除操作，但可以友好地帮忙设置下过期时期
    redis.call('pexpire', KEYS[1], ARGV[2]); 
    return 0; 
else 
    // 锁重入计数已为0：间接表明锁已释放了。直接删除掉锁，并广播解锁消息，去唤醒那些争抢过锁但还处于阻塞中的线程
    redis.call('del', KEYS[1]); 
    redis.call('publish', KEYS[2], ARGV[1]); 
    return 1;
end;
 
return nil;

```

## redis集群模式： 

### 单节点实例

### 主从模式（master/slaver）

+ 主从模式的作用：
    + 主从模式的一个作用是备份数据，这样当一个节点损坏（指不可恢复的硬件损坏）时，数据因为有备份，可以方便恢复。
    + 另一个作用是负载均衡，所有客户端都访问一个节点肯定会影响Redis工作效率，有了主从以后，查询操作就可以通过查询从节点来完成。

+ 对主从模式必须的理解（结论已经验证过，可以自行验证）：
    + 一个Master可以有多个Slaves
    + 默认配置下，master节点可以进行读和写，slave节点只能进行读操作，写操作被禁止
    + 不要修改配置让slave节点支持写操作，没有意义，原因一，写入的数据不会被同步到其他节点；原因二，当master节点修改同一条数据后，slave节点的数据会被覆盖掉
    + slave节点挂了不影响其他slave节点的读和master节点的读和写，重新启动后会将数据从master节点同步过来
    + master节点挂了以后，不影响slave节点的读，Redis将不再提供写服务，master节点启动后Redis将重新对外提供写服务。
    + master节点挂了以后，不会slave节点重新选一个master

+ 对主从模式的缺点： 
    + master节点挂了以后，redis就不能对外提供写服务了
    + 剩下的slave不能成为master

+ slaveof属性指向master

### sentinel模式（哨兵，守卫）

+ 作用： 
    + 解决主从模式的缺点，
    + 高可用性。
+ 实现方式： 
    + 当sentinel发现master节点挂了以后，sentinel就会从slave中重新选举一个master。
    + 当master节点重新启动后，它将不再是master而是做为slave接收新的master节点的同步数据
    + 一个sentinel或sentinel集群可以管理多个主从Redis。
    + 当主从模式配置密码时，sentinel也会同步将配置信息修改到配置文件中
    + sentinel监控的Redis集群都会定义一个名字，这个名字代表Redis集群
    + 客户端连接sentinel的ip和port，由sentinel来提供具体的可提供服务的Redis实现


### cluster模式

+ 作用
    + cluster的出现是为了解决单机Redis容量有限的问题，将Redis的数据根据一定的规则分配到多台机器



### 参考文章

+ [Redis真的那么好用吗？](http://www.redis.cn/articles/20181020002.html)
+ [Redis分布式锁完美方案](https://blog.csdn.net/asd051377305/article/details/108384490)

