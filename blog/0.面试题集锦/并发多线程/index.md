## 并发和并行都是什么意思

并发：宏观上，同一时刻多个任务或多个请求同时执行。微观上是多个线程快速交替进行，实际还是串行。

并行：多个处理器同时处理多个不同的任务

## 并发编程中的三个概念
+ 原子性
+ 可见性
+ 有序性


## 如何解决原子性、可见性、有序性问题

###  原子性：
+ 关键字Synchronized来保证代码块内的操作是原子

### 可见性：
+ volatile关键字其修饰的变量在被修改后可以立即同步到主内存，缓存了此变量的线程在使用之前都会从主内存刷新。
+ 通过缓存一致性协议来解决可见性问题。
+ synchronized和ﬁnal两个关键字也可以实现可见性 。
+ volatile也可以看作是轻量级的锁，在其内部使用了Lock指令来解决可见性问题。 


### 有序性：
+ volatile关键字通过增加内存屏障来解决有序性问题



## 线程和进程的区别

进程：指在系统中正在运行的一个应用程序；程序一旦运行就是进程；进程——资源分配的最小单位。

线程：系统分配处理器时间资源的基本单元，或者说进程之内独立执行的一个单元执行流。线程——程序执行的最小单位。


## 线程的几种状态以及相互转化

初始状态(new)，就绪状态(Runnable)，运行状态(Running)，阻塞状态(Blocked)，销毁状态(Dead)

+ 初始调用start方法，进入就绪
+ 就绪被cpu分配当时间片段进入运行状态
+ 运行调用yield方法，让出时间片段，进入就绪状态
+ 运行调用sleep/join/wait进入阻塞状态，不同的是sleep不释放锁，join和wait释放锁（join底层是wait）。
+ 阻塞调用notify/notifyall进入就绪状态
+ 程序正常运行结束退出,程序抛出未捕获的异常(Exceltion/Error),调用stop方法(可能会导致死锁，不推荐使用)进入销毁状态。

## 线程的实现方式

继承Thread

实现Runnable，可多实现

实现Callable，可多实现，还带返回值。可以获取一个Future对象，在该对象上调用get就可以获取到Callable返回的结果对象

## 线程池的实现方式

线程池是为了减少线程的创建和销毁。

newCachedThreadPool：

```java 

 public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }

```

newFixedThreadPool

```java 
public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }
```

newSingleThreadExecutor

```java 

  public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }

```

newScheduledThreadPool


```java 

    public ScheduledThreadPoolExecutor(int corePoolSize) {
        super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS,
              new DelayedWorkQueue());
    }

```

 🌟 ThreadPoolExecutor

 ```java 
ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler)


上面的4个封装线程池都是基于ThreadPoolExecutor对象创建
参数列表分别是：核心线程数，最大线程数，超时时间，时间单位，等待队列(缓存队列)，线程工厂，拒绝策略


 ```

### 线程池工作原理：

当任务来临，首先判断核心线程数是否到达，未到达直接运行任务，否则就判断等待队列是否满了，未满的话将任务放进等待队列，当等待队列满了，就判断是否到达最大线程数，如果未到达最大线程数，就直接创建线程，当线程数到达最大线程数，等待队列也满了，就执行拒绝策略。当所有的任务都执行完毕，线程等待超时时间没有新任务来临进行销毁线程，直到销毁到数量剩余核心线程数为止。


### 拒绝策略

当线程池的任务缓存队列已满并且线程池中的线程数目达到maximumPoolSize，如果还有任务到来就会采取任务拒绝策略。


（1）ThreadPoolExecutor.AbortPolicy 丢弃任务，并抛出RejectedExecutionException异常

（2）ThreadPoolExecutor.CallerRunsPolicy：该任务被线程池拒绝，由调用 execute方法的线程执行该任务。

（3）ThreadPoolExecutor.DiscardOldestPolicy ： 抛弃队列最前面的任务，然后重新尝试执行任务。

（4）ThreadPoolExecutor.DiscardPolicy，丢弃任务，也不抛出异常。

（5）自定义拒绝策略: 实现RejectedExecutionHandler，并重写rejectedExecution方法

    ```java 

    class AutoRejected implements RejectedExecutionHandler{
            @Override
            public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
                // dothing                
            }
        }

    ```




## 线程池的几种阻塞队列

+ ArrayBlockingQueue :由数组结构组成的有界阻塞队列。
  + 先进先出FIFO
  + 默认不公平。 创建公平：new ArrayBlockingQueue(1000,true)
+ LinkedBlockingQueue :由链表结构组成的有界阻塞队列。
  + 先进先出FIFO
  + 生产者和消费者端分别采用了独立锁来控制数据。
  + 默认是一个无限量大小的队列（Integer.MAX_VALUE）
+ PriorityBlockingQueue :支持优先级排序的无界阻塞队列。
  + 自定义实现compareTo方法来实现优先级排序
+ DelayQueue:使用优先级队列实现的无界阻塞队列。
  + 支持延时获取元素的无界则色队列
  + 只有当延时期到了才能从队列获取元素
+ SynchronousQueue:不存储元素的阻塞队列。
  + 每一个put操作都必须等待一个take操作，否则不能继续添加元素
  + 相当于一个传球手，自己本身不存储任何元素，
  + 吞吐量高于ArrayBlockingQueue和LinkedBlockingQueue
+ LinkedTransferQueue:由链表结构组成的无界阻塞队列。
  + transfer方法：如果当前有消费者正在等待接收元素(消费者使用take方法，或带时间限制的poll方法)，transfer 方法可以把生产者传入的元素立刻 transfer(传输)给消费者。如果没有消费者在等待接收元素，transfer 方法会将元素存放在队列的 tail 节点，并等到该元素被消费者消费了才返回。
  + tryTransfer方法，则是用来试探生产者传递的元素是否能直接传给消费者。如果没有消费者等待接收元素，则返回 false。和 transfer 方法的区别是 tryTransfer 方法无论消费者是否接收，方法立即返回。而 transfer 方法是必须等到消费者消费了才返回。
+ LinkedBlockingDeque:由链表结构组成的双向阻塞队列
  + 可以从队列的两端插入和移除元素



