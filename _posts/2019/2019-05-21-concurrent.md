---
layout: post
title: 读写锁ReentrantReadWriteLock
category: note
tags: [concurrent]
no-post-nav: true
---

读多写少的场景下，ReentrantLock就有点力不从心了，所以ReentrantReadWriteLock应运而生。ReentrantReadWriteLock采用读写分离的策略，允许多个线程同时获取读锁。内部维护了一个ReadLock和一个WriteLock，它们依赖Sync实现具体的功能。而Sync类继承自AQS，并且也提供了公平和非公平的实现。ReentrantReadWriteLock通过state的高16位表示读状态，也就是获取到读锁的次数；低16位表示获取到写锁的可重入次数。类图结构如下:

![ReentrantReadWriteLock类图](http://image.wyc1856.club/2019-08-01-09-53-53.png)
```
/** 偏移位 */
static final int SHARED_SHIFT = 16;
/** 共享锁(读锁)状态单位值(65536) */
static final int SHARED_UNIT = (1 << SHARED_SHIFT);
/** 共享锁线程最大个数(65535) */
static final int MAX_COUNT = (1 << SHARED_SHIFT) - 1;
/** 独占锁(写锁)掩码 */
static final int EXCLUSIVE_MASK = (1 << SHARED_SHIFT) - 1;

/** 返回读锁线程数 */
static int sharedCount(int c)  { return c >>> SHARED_SHIFT; }
/** 返回写锁可重入次数 */
static int exclusiveCount(int c) { return c & EXCLUSIVE_MASK; }
```

## 写锁的获取与释放(非公平策略)
> void lock()方法
>> 写锁是独占锁，如果当前没有线程获取到读锁和写锁(state=0)，则当前线程通过CAS操作更新state++尝试获取锁，成功则设置exclusiveOwnerThread为当前线程。如果当前已经有线程获取到读锁或写锁(state!=0)，然后判断w=exclusiveCount(state)是否为0，有线程获取了读锁(w==0)或者当前线程非写锁的持有者就挂起当前线程，创建独占节点(Node.EXCLUSIVE)，加入AQS阻塞队列。

> boolean tryLock()方法
>> 尝试获取写锁，和非公平策略的lock()方法类似，未获得锁线程不会阻塞。

> boolean tryLock(long timeout, TimeUnit unit)方法
>> 与tryLock()方法类似，只是多了超时时间设置。

> void unLock()方法
>> 尝试释放锁，state的高16位一定都为0，所以直接操作state--即可，判断state是否为0，为0则释放锁(设置exclusiveOwnerThread为null)。

## 读锁的获取与释放(非公平策略)
> void lock()方法
>> 首先判断当前是否有其他线程获取了写锁(exclusiveCount(state) != 0 && getExclusiveOwnerThread() != current)，直接返回-1。然后调用AQS的doAccquireShared方法把当前线程放入AQS阻塞队列。如果当前线程已经获取了写锁，则也能继续获取读锁。但是要注意，读锁处理事情完毕后，要记得把读锁和写锁都释放掉，不能只释放写锁（没明白为啥要这样做）。如果当前没有线程获取写锁(exclusiveCount(state) == 0)，则计算出读锁计数(r = sharedCount(state))，然后尝试获取锁(逻辑比较复杂，暂时先这样吧)。

> boolean tryLock()
>> 尝试获取读锁，逻辑和lock()类似，区别是未获取到锁线程并不会阻塞。

> boolean tryLock(long timeout, TimeUnit unit)方法
>> 与tryLock()方法类似，多了超时时间设置，对线程中断操作响应。

> void unlock()
>> 尝试释放读锁，循环尝试通过CAS操作state高16位减一，CAS操作成功则判断当前state是否为0，为0则说明当前无读线程占用读锁。tryReleaseShared方法返回true，然后调用doReleaseShared方法释放一个由于获取写锁而被阻塞的线程。