---
layout: post
title: 独占锁ReentrantLock
category: note
tags: [concurrent]
no-post-nav: true
---

ReentrantLock是可重入的独占锁，内部通过Sync(继承AQS)提供了NonfairSync非公平锁和FairSync公平锁的实现。state状态值表示线程获取该锁的可重入次数，在默认情况下，state的值为0表示当前锁没有被任何线程持有，类图结构如图：

![ReentrantLock类图](http://image.wyc1856.club/2019-07-31-14-56-59.png)

> void lock()方法
>> 非公平锁：首先通过CAS操作尝试修改状态0->1，成功则设置exclusiveOwnerThread为当前线程。失败则调用acquire()方法，调用重写的tryAcquire()方法获取锁。当前已有线程获得锁了，则判断锁的持有者是否为当前线程，为当前线程则state++，否则挂起当前线程，加入AQS队列。

>> 公平锁：直接调用acquire()方法，调用重写的tryAcquire()方法。首先判断state是否为0，为0则调用hasQueuedPredecessors()判断AQS阻塞队列是否存在阻塞节点，不存在才会通过CAS操作尝试获取锁，成功获得锁后设置exclusiveOwnerThread为当前线程，中途其他情况则直接挂起当前线程，加入AQS队列。不为0则判断当前线程是否为锁的持有者，是则更新state++，不是则挂起当前线程，加入AQS队列。

> boolean tryLock()方法
>> 尝试获取锁，使用非公平策略，如果当前该锁没有被其他线程持有，则当前线程获取该锁并返回true，否则返回false。注意，该方法不会引起当前线程阻塞。

> boolean tryLock(long timeout, TimeUnit unit)方法
>> 尝试获取锁，与tryLock()的不同之处在于，它设置了超时时间，如果超时时间到了还没获得该锁则返回false。

> void unlock()方法
>> 尝试释放锁，更新state--，更新后的state为0则释放该锁(设置exclusiveOwnerThread为null)，否则仅仅减1而已。如果当前线程没有持有该锁而调用了该方法则会抛出IllegalMonitorStateException异常。