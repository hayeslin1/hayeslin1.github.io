# 数据库 

## 数据库的三大范式

第一范式：每个列都不可以再拆分。

第二范式：在第一范式的基础上，非主键列完全依赖于主键，而不能是依赖于主键的一部分。

第三范式：在第二范式的基础上，非主键列只依赖于主键，不依赖于其他非主键。

BC范式：在第三范式的基础上，消除任何属性（主属性和非主属性）对关系键的部分函数依赖和传递函数依赖。

## 事务的特点：

+ Atomicity原子性

+ Consistency一致性

+ Isolation隔离性

+ Durability永久性

## 事务并发会产生的问题

脏读：一个事务读取了另一个未提交事务的数据。需要注意的是这里针对的是数据本身，可以理解为针对单笔数据。

不可重复读： 一个事务进行读取，分别读取到了不同的数据。需要注意的是这里针对的是数据本身，可以理解为针对单笔数据，重点是对数据的修改和删除，所以对行加锁就可以解决。

幻读： 一个事务进行读取，分别读取到了不同的数据。需要注意的是这里针对的是数据条数，可以理解为针对多笔数据是个数据集，重点是对数据的新增，所以对表加锁就可以解决。

## 事务的隔离级别

+ DEFAULT
> 使用数据库本身使用的隔离级别
ORACLE（读已提交） MySQL（可重复读）

+ Read uncommitted
> 读未提交，顾名思义，就是一个事务可以读取另一个未提交事务的数据。

+ Read committed
> 读提交，顾名思义，就是一个事务要等另一个事务提交后才能读取数据。
解决了脏读，但不能解决不可重复读和幻读。

+ Repeatable read
> 重复读，就是在开始读取数据（事务开启）时，不再允许修改操作
解决了不可重复读，但不能解决幻读。

+ Serializable 序列化
> Serializable 是最高的事务隔离级别，在该级别下，事务串行化顺序执行，可以避免脏读、不可重复读与幻读。但是这种事务隔离级别效率低下，比较耗数据库性能，一般不使用。


❌ 会产生的问题 ，  ✅  解决的问题

| 隔离级别  | 脏读  | 不可重复读  |幻读|
|---|---|---|---|
|  Read uncommitted | ❌ | ❌ |❌|
|  Read committed |  ✅ | ❌ | ❌|
|  Repeatable read |  ✅ |  ✅ |❌|
|  Serializable | ✅ |  ✅ |✅|


幻读： 使用MVCC解决


## MVCC

> MVCC的实现，通过保存数据在某个时间点的快照来实现的。这意味着一个事务无论运行多长时间，在同一个事务里能够看到数据一致的视图。根据事务开始的时间不同，同时也意味着在同一个时刻不同事务看到的相同表里的数据可能是不同的。

+ 行数据都存在一个版本，每次数据更新时都更新该版本。
+ 修改时Copy出当前版本随意修改，各个事务之间无干扰。
+ 保存时比较版本号，如果成功（commit），则覆盖原记录；失败则放弃copy（rollback）

[参考文档MVCC](https://www.cnblogs.com/shujiying/p/11347632.html)

## mysql的存储引擎

 MyISAM 和InnoDB区别
|   | MyISAM  | InnoDB  | 
|---|---|---|
|事务|不支持|支持|
|外建|不支持|支持|
|行锁|不支持|支持|
|表锁|支持|支持|
|唯一索引|没有|必须有（没有的话使用因此的rowid）|
|count|直接返回|扫全表返回|
|是否聚集索引|非聚集索引|聚集索引|
|数据存放|索引+引用地址->数据<br>理解为B+树的末端是数据存放的索引地址指向数据|索引+数据<br> 理解为B+树的末端是数据|
|文件|定义文件Frm+索引文件MyI+数据文件MyD|定义文件Frm+数据文件IBD|
|灾后恢复|相对困难|相对容易|
|delete语句|删表重建|一行行删除|
|内存|占用少|对比：多|
|特点|增、查多比较合适|更新多合适|


## 聚簇索引和非聚簇索引

表记录的排序顺序和索引的排序
1. 顺序一致： 聚簇索引
2. 顺序不一致： 非聚簇索引

> 一个表只能有一个聚集索引，可以有多个非聚集索引


## B+树， B树，红黑树的区别

+ 红黑树是一种自平衡二叉搜索树,最多只有两个节点。在插入和删除数据的时候需要进行旋转操作来保证树的平衡，而且在大规模数据存储的时候，红黑树往往出现由于树的深度过大而造成磁盘IO读写过于频繁，进而导致效率低下的情况。
+ B树和B+树都是一种多路查找树，可以有多个子节点，相比红黑树降低树的高度。
+ B树则所有节点都带有指向记录（数据）的指针（ROWID），B+树中只有叶子节点会带有指向记录(数据)的指针（ROWID，B+树非叶子节点只存放关键字，这样一个块可以容纳更多的关键字，可以快速的定位更多的叶子节点。
+ 范围查找：B+树所有叶子节点都是通过指针前后相连，适合于范围查询。B树则需要在叶子节点和非叶子节点不停的往返遍历查询
+ 获取所有数据：B树需要通过中序遍历所有节点才能获取所有数据，而B+树则只需要遍历叶子节点即可。

## 联合索引， 最左前缀， 回表，索引覆盖， 索引下推

+ 首先根据联合索引中最左边的、也就是第一个字段进行排序，在第一个字段排序的基础上，再对联合索引中后面的第二个字段进行排序，依此类推
+ 回表： 先通过普通索引找到叶子节点的主键，再通过主键索引树，找到叶子节点的数据（或数据指针）
+ 索引覆盖：通过普通索引能一次把所有字段全部查出来， 不需要回表查询
+ 索引下推： 联合索引中，所有的字段同时当作条件进行过滤查询，减少了回表的次数。

## explain 解释sql执行计划

+ type=const: 通过索引一次查到
+ type=all： 全表扫描
+ type=eq_ref： 唯一索引 
+ type=ref： 非唯一索引扫描，如联合索引
+ type=range： 检索了范围内的行


+ key=primary： 使用了主键suoyin 
+ key=null： 没有使用索引


### 字段详解
+ id

> 一组数字 

> id相同，表示顺序执行，至上而下

> id值越大优先级越高，越先被执行 

+ select_type

> 查询的类型，主要是用于区分普通查询、联合查询、子查询等复杂的查询

``` xml 
1、SIMPLE：简单的select查询，查询中不包含子查询或者union 
2、PRIMARY：查询中包含任何复杂的子部分，最外层查询则被标记为primary 
3、SUBQUERY：在select 或 where列表中包含了子查询 
4、DERIVED：在from列表中包含的子查询被标记为derived（衍生），mysql或递归执行这些子查询，把结果放在零时表里 
5、UNION：若第二个select出现在union之后，则被标记为union；若union包含在from子句的子查询中，外层select将被标记为derived 
6、UNION RESULT：从union表获取结果的select 
```

+ type: 访问类型

> system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > ALL

+ key

> 实际使用的索引，如果为NULL，则没有使用索引。 

+ key_len

> 表示索引中使用的字节数，查询中使用的索引的长度（最大可能长度），并非实际使用长度，理论上长度越短越好。

+ ref

> 显示索引的那一列被使用了，如果可能，是一个常量const。

+ rows

> 根据表统计信息及索引选用情况，大致估算出找到所需的记录所需要读取的行数

+ Extra

> Using index： 表示相应的select操作中使用了覆盖索引（Covering Index）
> 
> Using where ： 使用了where过滤


+ [参考文档（SQL执行计划详解explain）](https://www.cnblogs.com/yhtboke/p/9467763.html)


## 主从复制配置

1. 修改主从库的配置文件my.cnf

vi /etc/my.cnf

主库：

```shell

[mysqld]
# 表示开启二进制日志
log-bin=mysql-bin
# 服务器的唯一id
server-id=1 
# 设置需要同步的数据库
binlog-do-db=user_db
# 屏蔽系统库同步
binlog-ignore-db=mysql
binlog-ignore-db=information_scheme
binlog-ignore-db=performance_schema


```

从库

``` shell

[mysqld]
# 表示开启二进制日志
log-bin=mysql-bin
# 服务器的唯一id
server-id=2

# 设置需要同步的数据库
replicate_wild_do_table=user_db.%

# 屏蔽系统库同步
replicate_wild_ignore_table=mysql.%
replicate_wild_ignore_table=information_scheme.%
replicate_wild_ignore_table=performance_schema.%

``` 

2. 创建并授权主从复制的专用账号

```shell

# 主库创建slave/db_sync 账号

# 确认位点，记录文件名和位点
mysql> show master status ; 

file ： mysql-bin.00002 // 文件名
position: 154 //位点


```
3. 设置从库同步数据

```shell

mysql> stop slave;

mysql> change master to 
    master_host = 'localhost'
    master_user = 'db_sync'
    master_password = 'db_sync'
    master_file = 'mysql-bin.00002'
    master_log_pos = '154'

mysql> start slave;

mysql> show slave status ;

slave_io_running: yes
slave_sql_running: yes

# 两个都是yes才行


```





## 主从复制原理

+ 分为同步复制和异步复制，大部分都采用异步复制
+ 主master任何修改操作都会写到二进制日志binlog中
+ 从服务器启动一个io thread，（实际上主服务器的客户端进程）
+ 连接到主服务器请求读取二进制日志，把读取到二进制日志写到本地的realylog里
+ 从服务器上的启动一个sql thread，定时检查realylog，如有更改就在本机执行一边



## 分库分表

分库分表分为垂直切分和水平切分。



## 分表后的ID怎么保证唯一性的呢？

设置全局ID， 可采用twitter的雪花算法或其他很多全局ID算法。


## 日志系统：redo log、binlog、undo log 区别与作用

+ 重做日志（redo log）

> 确保事务的持久性。防止在发生故障的时间点，尚有脏页未写入磁盘，在重启mysql服务的时候，根据redo log进行重做，从而达到事务的持久性这一特性。

+ 回滚日志（undo log）

> 保存了事务发生之前的数据的一个版本，可以用于回滚，同时可以提供多版本并发控制下的读（MVCC），也即非锁定读

+ 二进制日志（binlog）：

> 用于复制，在主从复制中，从库利用主库上的binlog进行重播，实现主从同步。 用于数据库的基于时间点的还原。


