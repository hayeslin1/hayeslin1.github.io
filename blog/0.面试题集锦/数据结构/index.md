# ArrayList

1. ArrayList extends AbstractList implements List, RandomAccess, Cloneable, Serializable
   1. 实现了List，它是一个元素有序(按照插入的顺序维护元素顺序)、可重复、可以为null的集合
   2. 同时还实现了 RandomAccess、Cloneable、Serializable 接口，所以ArrayList 是支持快速访问、复制、序列化的。
2. 底层以动态array数组的形式实现的 
   1. 默认容量为10 
   2. 扩容：old + old >> 1 ( old + old 除以 2 的1次幂 )   = 3/2倍
   3. 最大：Integer.MAX_VALUE - 8
3. 扩容是以Arrays.copyof()的方式拷贝的， 非常耗时
   1. 查询快， 增删慢

# LinkedList

1. LinkedList extends AbstractSequentialList implements List ，Deque，Cloneable，Serializable
   1. 继承自AbstractSequentialList： 只支持按顺序访问，而不像 AbstractList 那样支持随机访问。不支持 RandomAccess
   2. 实现了List，它是一个元素有序(按照插入的顺序维护元素顺序)、可重复、可以为null的集合
   3. 实现了Deque， 双端队列，支持在两端插入和删除元素
   4. 实现了Cloneable，Serializable 支持复制、序列化。
2. 是一个双向链表没有初始化大小，也没有扩容的机制 ， 最大的容量为Integer.MAX_VALUE
3. 新增/删除元素就是移动一下指针， 查询的时候只能从头或者尾一个一个查
   1. 增删快， 查询慢


# HashMap

1. HashMap<K,V> extends AbstractMap<K,V>  implements Map<K,V>, Cloneable, Serializable
2. 底层是数组+链表+红黑树
   1. 链表长度特别长的时候查询特别慢，所以JDK8之后转为红黑树
      1. 红黑树每个节点只有红色和黑色
      2. 根结点是黑色
      3. 每个叶子节点都是黑色空节点
      4. 从根结点到叶子节点不能出现两个连续的红色节点
      5. 从任一节点出发， 到下面子节点的路径包含相同数据的黑色节点
   2. 红黑树查询就相对快很多
3. 链表长度超过8，转化为红黑树，减少到6就退化为链表
4. 转为红黑树前提是：数组容量超过64
5. 初始化大小是 16 ，扩容因子默认0.75（可以指定初始化大小，和扩容因子）
6. 扩容机制： 当前大小 和 当前容量 的比例超过了 扩容因子，就会扩容，扩容后大小为 一倍。  
7. JAVA Hashmap的死循环及Java8的修复
   1. HashMap是非线程安全的，线程安全的应该用ConcurrentHashMap。
   2. JDK1.7是采用的头插法，在多线程环境下有可能会使链表形成环状，从而导致死循环。
   3. JDK1.8做了改进，用的是尾插法，不会产生死循环。
8. **put过程**
	1. 对Key求Hash，然后计算下标
	2. 如果没有碰撞放入桶中（碰撞的意思是计算的hash值一样，需要放进一个桶里）
	3. 如果发生了碰撞就存为链表
	4. 当总容量>64,链表的长度>8，就转为红黑树存储。当链表长度<6，再换成链表。
	5. 如果hash值和equals值都相等，说明是相同的key，则替换value
	6. 如果桶满了（容量16*加载因子0.75）， 则需要进行resize（扩容2倍，之后进行重排）
9. 查询的效率取决于： 散列函数，冲突处理机制，和装载因子。 
10. 解决碰撞：开放定址法，再散列法，链地址法，建立公共溢出区
11. 常用的散列函数： 直接寻址法，数字分析法，平方取中法，折叠法， 随机数法，除留余数法，乘法取整法。

# ConcurrentHashMap跟HashMap，HashTable的对比
1. HashTable和HashMap的实现原理几乎一样
2. HashMap不是线程安全：HashTable是线程安全的：
3. HashTable线程安全给整个哈希表加了一把大锁
4. ConcurrentHashMap所采用的"分段锁"思想
5. ConcurrentHashMap为table数组＋单向链表＋红黑树的结构.

# 各数组结构区分
1. **list： 双向链表。** 
2. vector： 动态数组
3. deque：双端队列
4. set： 底层是红黑树
5. map： 底层红黑树

# 二叉树 
1. 二叉树： 度最大为2（左子树，右子树）
2. 最优二叉树（哈夫曼树）： 该树的带权路径长度达到最小
3. 满二叉树：所有非叶子节点度都是2，且叶子节点再同一个层次
4. 完全二叉树： 与满二叉树的前m层的结构相同
5. 平衡二叉树： 左子树和右子树高度**大致**相同，高度差小于等于1
	1. AVL树
	2. 红黑树
