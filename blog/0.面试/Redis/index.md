# Redis 






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
