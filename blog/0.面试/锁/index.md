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




## 可重入锁是如何实现重入的

## Synchronized 同步锁 

## Lock 接口

## Semaphore信号量

## CountDownLatch

## CyclicBrrier




