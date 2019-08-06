---
title: 并发编程 - 显式锁和AQS
tags:
  - 并发编程
---

显式锁和AQS

## 显式锁

Lock接口

```java

 	private Lock lock = new ReentrantLock();
    private int count;
    // 一定要在finally中释放锁
    public void increament() {
        this.lock.lock();

        try {
            ++this.count;
        } finally {
            this.lock.unlock();
        }

    }
```

- 可重入锁ReentrantLock(默认非公平锁 构造方法可控制是否是公平锁)

	采用一个计数器，每重入一次次数加一，释放一个锁次数减一

- 读写锁ReentrantReadWriteLock

	同一时刻允许多个读线程同时访问，但是写线程访问的时候，所有的读和写都被阻塞，最适宜与读多写少的情况

- Condition接口

	阻塞一个线程await()，唤醒是最好使用singal().wait()阻塞线程使用notfiyAll()唤醒。


<!-- more -->  

## AQS AbstractQueuedSynchronizer

AQS是AbstractQueuedSynchronizer的简称。AQS提供了一种实现阻塞锁和一系列依赖FIFO等待队列的同步器的框架，如下图所示。AQS为一系列同步器依赖于一个单独的原子变量（state）的同步器提供了一个非常有用的基础。子类们必须定义改变state变量的protected方法，这些方法定义了state是如何被获取或释放的。鉴于此，本类中的其他方法执行所有的排队和阻塞机制。子类也可以维护其他的state变量，但是为了保证同步，必须原子地操作这些变量。

![AQS](http://image.tupelo.top/53727-ae36db58241c256b.png)

- 设计模式
	
	使用了模板方法设计模式，模板方法有：		
	独占式获取：		
		- accquire
		- acquireInterruptibly
		- tryAcquireNanos
	共享式获取
		- acquireShared
		- acquireSharedInterruptibly
		- tryAcquireSharedNanos
	独占式释放锁
		- release
	共享式释放锁
		- releaseShared



- state状态

  AbstractQueuedSynchronizer维护了一个volatile int类型的变量，用户表示当前同步状态。volatile虽然不能保证操作的原子性，但是保证了当前变量state的可见性。state的访问方式有三种:

``` java
getState()
setState()
// CAS
compareAndSetState() 
```

- 自定义资源共享方式

AQS定义两种资源共享方式：Exclusive（独占，只有一个线程能执行，如ReentrantLock）和Share（共享，多个线程可同时执行，如Semaphore/CountDownLatch）。

- AQS中的数据结构-节点和同步队列

	竞争失败的线程会打包成Node放到同步队列，Node可能的状态里：
	
	- CANCELLED：线程等待超时或者被中断了，需要从队列中移走
	- SIGNAL：后续的节点等待状态，当前节点，通知后面的节点去运行
	- CONDITION :当前节点处于等待队列
	- PROPAGATE：共享，表示状态要往后面的节点传播，0表示初始状态

- 独占式同步状态获取与释放


![](http://image.tupelo.top/%E5%9B%BE%E7%89%87%201.png)



















