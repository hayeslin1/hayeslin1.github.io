## Atomic关键字 和 Volatile关键字

volatile是Java提供的一种轻量级的同步机制。同synchronized相比（synchronized通常称为重量级锁），volatile更轻量级。

+ 1）保证了不同线程对这个变量进行操作时的可见性，即一个线程修改了某个变量的值，这新值对其他线程来说是立即可见的。

    + 使用CPU的一致性协议来保证可见性

+ 2）禁止指令重排序。
    
    + 代码前后加入了内存屏障来保证有序性 

+ 3）不保证原子性。原子性使用如下方式：

    + 使用单线程可以保证原子性
    + 也可以使用 synchronize 或者是锁的方式来保证原子性。
    + 还可以用 Atomic 包中 AtomicInteger 来替换 int，它利用了 CAS 算法来保证了原子性。

Atomic关键字:是原子操作类，AtomicInteger、AtomicLong、AtomicBoolean、AtomicReference<T>。 

+ 采用Lock-Free算法替代锁+原子操作指令实现并发情况下资源的安全、完整、一致性， 非阻塞式的并发问题。


## 可重入锁是如何实现重入的

就是可重复进入的锁，或者叫做递归锁。

锁的对象维护了一个state和一个threadName，加锁的时候state=1，threadName=当前线程，如果再次获取锁，判断是不是当前线程，如果是state+1，如果不是，则需要等待。
释放锁的时候state-1，直到减到为0，让其他阻塞的线程重新获取到可争抢的权利。  

## ThreadLocal

是线程的本地存储，在每个线程的内部都会创建一个ThreadLocalMap，每个线程都访问自己内部的value，



## Synchronized 同步锁 

## Lock 接口

## Semaphore信号量

## CountDownLatch

## CyclicBrrier




