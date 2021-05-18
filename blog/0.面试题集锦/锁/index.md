## Atomic关键字 和 Volatile关键字

volatile是Java提供的一种轻量级的同步机制。同synchronized相比（synchronized通常称为重量级锁），volatile更轻量级。

+ 1）保证了不同线程对这个变量进行操作时的可见性，即一个线程修改了某个变量的值，这新值对其他线程来说是立即可见的。

    + 使用CPU的一致性协议来保证可见性

+ 2）禁止指令重排序。
    
    + 代码前后加入了内存屏障(Memnory Barrier)来保证有序性 

+ 3）不保证原子性。原子性使用如下方式：

    + 使用单线程可以保证原子性
    + 也可以使用 synchronize 或者是锁的方式来保证原子性。
    + 还可以用 Atomic 包中 AtomicInteger 来替换 int，它利用了 CAS 算法来保证了原子性。

Atomic关键字:是原子操作类，AtomicInteger、AtomicLong、AtomicBoolean、AtomicReference<T>。 

+ 采用Lock-Free算法替代锁+原子操作指令实现并发情况下资源的安全、完整、一致性， 非阻塞式的并发问题。
+ 效率高于Synchronized 


## 可重入锁是如何实现重入的

就是可重复进入的锁，或者叫做递归锁。

锁的对象维护了一个state和一个threadName，加锁的时候state=1，threadName=当前线程，如果再次获取锁，判断是不是当前线程，如果是state+1，如果不是，则需要等待。
释放锁的时候state-1，直到减到为0，才能让其他阻塞的线程重新获取到可争抢的权利。

## 死锁

多个线程在允许过程中，因争夺资源而造成的一种僵局，处于这种僵局的状态下，若无外力的作用，他们都将无法推进。

### 产生死锁的原因： 
+ 系统资源不足
+ 进程间推进不当
+ 资源分配不均

### 死锁产生的必要条件

+ 互斥条件
+ 不可剥夺条件
+ 请求与保持条件
+ 循环等待条件

## ThreadLocal

是线程的本地存储，在每个线程的内部都会创建一个ThreadLocalMap，每个线程都访问自己内部的value，

ThreadLocal类型的本地变量是存放在具体的线程空间上，其本身相当于一个装载本地变量的工具壳，通过set方法将value添加到调用线程的threadLocals中，当调用线程调用get方法时候能够从它的threadLocals中取出变量。如果调用线程一直不终止，那么这个本地变量将会一直存放在他的threadLocals中，所以不使用本地变量的时候需要调用remove方法将threadLocals中删除不用的本地变量。

## Synchronized 同步锁 

可以作用于方法，代码块。 属于独占式的悲观锁，同时属于可重入锁， 非公平锁。

#### 核心组件： 

+ Wait Set: 调用了wait方法被阻塞的线程放置在这里
+ Contention List ： 竞争队列，所有请求锁的线程都被放在这里
+ Entry List ：在Contention List里有资格成为候选资源的线程被移动到这里。
+ onDeck： 任意时刻，最多只有一个线程成为候选线程，该线程称为onDeck
+ Owner 已经获取到资源的线程叫做Owner

#### Synchronized的实现过程

+ 1. JVM每次从队列的尾部取出一个数据用于锁竞争候选者(OnDeck)，但是并发情况下，ContentionList 会被大量的并发线程进行 CAS 访问，为了降低对尾部元素的竞争，JVM 会将一部分线程移动到 EntryList 中作为候选竞争线程。
+ 2. Owner 线程会在 unlock 时，将 ContentionList 中的部分线程迁移到 EntryList 中，并指定EntryList 中的某个线程为 OnDeck 线程(一般是最先进去的那个线程)。
+ 3. Owner 线程并不直接把锁传递给 OnDeck 线程，而是把锁竞争的权利交给 OnDeck，OnDeck 需要重新竞争锁。这样虽然牺牲了一些公平性，但是能极大的提升系统的吞吐量，在JVM 中，也把这种选择行为称之为“竞争切换”。
+ 4. OnDeck 线程获取到锁资源后会变为 Owner 线程，而没有得到锁资源的仍然停留在 EntryList中。如果 Owner 线程被 wait 方法阻塞，则转移到 WaitSet 队列中，直到某个时刻通过 notify或者 notifyAll 唤醒，会重新进去 EntryList 中。
+ 5. 处于 ContentionList、EntryList、WaitSet 中的线程都处于阻塞状态，该阻塞是由操作系统来完成的(Linux 内核下采用 pthread_mutex_lock 内核函数实现的)。
+ 6. Synchronized 是非公平锁。 Synchronized 在线程进入 ContentionList 时，等待的线程会先尝试自旋获取锁，如果获取不到就进入 ContentionList，这明显对于已经进入队列的线程是不公平的，还有一个不公平的事情就是自旋获取锁的线程还可能直接抢占 OnDeck 线程的锁资源。参考:https://blog.csdn.net/zqz_zqz/article/details/70233767

+ 7. 每个对象都有个 monitor 对象，加锁就是在竞争 monitor 对象，代码块加锁是在前后分别加上 monitorenter 和 monitorexit 指令来实现的，方法加锁是通过一个标记位来判断的
+ 8. synchronized 是一个重量级操作，需要调用操作系统相关接口，性能是低效的，有可能给线程加锁消耗的时间比有用操作消耗的时间更多。
+ 9. Java1.6，synchronized 进行了很多的优化，有适应自旋、锁消除、锁粗化、轻量级锁及偏向锁等，效率有了本质上的提高。在之后推出的 Java1.7 与 1.8 中，均对该关键字的实现机理做了优化。引入了偏向锁和轻量级锁。都是在对象头中有标记位，不需要经过操作系统加锁。
+ 10. 锁可以从偏向锁升级到轻量级锁，再升级到重量级锁。这种升级过程叫做锁膨胀;
+ 11. JDK 1.6 中默认是开启偏向锁和轻量级锁，可以通过-XX:-UseBiasedLocking 来禁用偏向锁。

## Lock 接口

+ void lock(): 加锁，如果无法加锁则处于阻塞状态，直到加锁成功
+ boolean tryLock():尝试加锁，成功true，否则false
+ boolean tryLock(long timeout ,TimeUnit unit): 在规定时间内尝试加锁，成功true，否则false
+ void unlock():释放锁
+ Condition newCondition(): 条件对象，获取等待通知组建。
+ boolean isFair(): 该锁是否是公平锁
+ boolean isLock(): 是否处于加锁状态
+ void lockInterruptibly(): 如果当前线程未被中断，获取锁

## ReentrantLock 继承自Lock并实现接口中的方法

也是一种可重入锁，能完成Synchronized所能完成的所有工作，还提供了可响应中断锁，可轮询加权锁，定时锁，公平锁， 多个锁等。

### Condition 类和 Object 类锁方法区别区别
+ Condition 类的 awiat 方法和 Object 类的 wait 方法等效
+ Condition 类的 signal 方法和 Object 类的 notify 方法等效
+ Condition 类的 signalAll 方法和 Object 类的 notifyAll 方法等效
+ ReentrantLock 类可以唤醒指定条件的线程，而 object 的唤醒是随机的


### synchronized 和 ReentrantLock 的区别

+ synchronized是JVM级别的锁，ReentrantLock是API级别的
+ synchronized加锁和解锁的过程是JVM来操作实现，进入相关代码自动加锁，执行完毕自动解锁。ReentrantLock是使用lock获取锁，unlock解锁，为避免死锁，通常unlock需要在finally中完成。
+ synchronized是不可中断的锁，ReentrantLock可响应中断，可轮回。
+ ReentrantLock可实现公平锁，new ReentrantLock(true)
+ ReentrantLock可以通过Condition绑定多个条件。
+ 底层实现：synchronized 是同步阻塞的悲观并发策略。  ReentrantLock是同步非阻塞的乐观并发策略
+ ReentrantLock有更灵活的获取锁，释放锁，查看锁状态等。


## CAS ：（ CAS(Compare And Swap/Set)比较并交换）

CAS算法是一种乐观锁的实现。比较并交换。

CAS会产生ABA问题，ABA问题是另一个线程将V由A更新成B，之后又更新成A，此刻对于当前线程是V还是A
解决ABA问题，可以增加一个version版本号，每次更新version+1。

## Semaphore 信号量

用于控制同时访问线程的个数。

+ void acquire(): 用于获取一个许可，拿不到就阻塞
+ void acquire(int permits)： 用于一次获取n个许可，拿不到就阻塞
+ void release() 用于释放一个许可，释放之前必须获取许可
+ void release(int permits) 用于一次释放n个许可，释放之前必须获取许可

+ boolean tryAcquire(): 尝试获取一个许可，成功true，不成功false
+ boolean tryAcquire(long timeout , TimeUnit unit) ：尝试一次获取一个许可，若再指定时间内获取成功返回true，不成功false
+ boolean tryAcquire(int permits)尝试获取n个许可，成功true，不成功false
+ boolean tryAcquire(int permits，long timeout , TimeUnit unit)尝试一次获取n个许可，若再指定时间内获取成功返回true，不成功false
+ int availablePermits()： 返回可用许可的数量

```java 

public class Worker extends Thread {

    private Semaphore semaphore;

    public Worker(Semaphore semaphore) {
        this.semaphore = semaphore;
    }


    public static void main(String[] args) throws IOException {
        int countWorker = 15;
        int countMachine = 5;
        final Semaphore semaphore = new Semaphore(countMachine);

        for (int i = 0; i < countWorker; i++) {
            new Worker(semaphore).start();
        }

    }


    @Override
    public void run() {
        try {
            semaphore.acquire();
            System.out.println("工人" + Thread.currentThread().getName() + "获取到机器的使用权");
            System.out.println("剩余可用许可证："+semaphore.availablePermits());
            Thread.sleep(1000);
            semaphore.release();
            System.out.println("工人" + Thread.currentThread().getName() + "使用完毕。");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}


```



## CountDownLatch （线程计数器）

位于JUC包下， 实现类似与计数器的功能，

使用代码：
```java 

public class CountDownLatchDemo extends Thread {

    private CountDownLatch latch;

    public CountDownLatchDemo(CountDownLatch latch) {
        this.latch = latch;
    }

    public static void main(String[] args) throws IOException {
        int count = 5;
        final CountDownLatch latch = new CountDownLatch(count);

        for (int i = 0; i < count; i++) {
            new CountDownLatchDemo(latch).start();
        }
        System.out.println("等待 " + count + " 个子线程执行完毕...");
        try {
            latch.await();
            System.out.println(count + " 个子线程已经执行完毕");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }


    @Override
    public void run() {
        System.out.println("子线程" + Thread.currentThread().getName() + "正在执行");
        try {
            Thread.sleep(500);
            System.out.println("子线程" + Thread.currentThread().getName() + "执行完毕");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            latch.countDown();
        }
    }
}


```



## CyclicBarrier (回环栅栏) 

当一组线程全部到达某个状态之后再同时执行。


```java 

public class CyclicBarrierDemo extends Thread {

    private CyclicBarrier cyclicBarrier;

    public CyclicBarrierDemo(CyclicBarrier cyclicBarrier) {
        this.cyclicBarrier = cyclicBarrier;
    }


    public static void main(String[] args) throws IOException {
        int count = 5;
        final CyclicBarrier latch = new CyclicBarrier(count);

        for (int i = 0; i < count; i++) {
            new CyclicBarrierDemo(latch).start();
        }
        System.out.println("等待 " + count + " 个子线程到达某个状态...");
    }


    @Override
    public void run() {
        System.out.println("子线程" + Thread.currentThread().getName() + "正在执行");
        try {
            Thread.sleep(500);
            cyclicBarrier.await();
        } catch (InterruptedException | BrokenBarrierException e) {
            e.printStackTrace();
        }
        System.out.println("子线程" + Thread.currentThread().getName() + "到达某个状态");
    }
}

```






## ConcurrentHashMap 分段锁

ConcurrentHashMap内部细分了若干小的HashMap，每一个被称为段（Segment）。

当需要向ConcurrentHashMap里put元素的时候会先计算key的hash判断得到应该放在哪个段下，然后对段加锁，而不是整个HashMap加锁。这样就提高了锁的并发粒度。



## AQS： AbstractQueuedSynchronizer 抽象的队列式的同步器

AQS定义了一套多线程访问共享资源的同步器框架

常用的 ReentrantLock/Semaphore/CountDownLatch等都是依赖于它实现的。 

AQS只定义了一个接口，具体的实现交由自定义同步器去实现了，AQS维护了一个volatile int state和一个FIFO线程等待队列。 AQS有两种共享方式exclusive独占式（如ReentrantLock）和share共享式（如Semaphore/CountDownLatch）

### 同步器的实现是ABS核心：

独占锁实现：tryAcquire-tryRelease 

共享锁实现：tryAcquireShared-tryReleaseShared 


+ 已ReentrantLock为例，state初始化为0，表示未锁定状态。线程lock时会调用tryAcquire独占该锁，state+1。此后其他线程在tryAcquire就会失败，直到线程unlock，state=0（释放了锁），其他线程才有机会获取该锁。当前线程释放之前也是可以重复获取此锁的，state会一直+1，这就是可重入的概念。需要注意，获取多少次就要释放多少次，这样才能保证state=0，不至于进入死锁状态。
+ 已CountDownLatch为例，任务分为N个子线程去执行，state也初始化为N（state初始化的N和线程数个数是相等的），这N个线程数是并行执行的，每个线程调用countDown()一次，state会CAS减1，等state=0，会unpark()主调用线程，然后主调用线程会从await()函数返回，继续后续的动作。


