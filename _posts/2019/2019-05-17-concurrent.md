---
layout: post
title: 抽象同步队列（AQS）
category: note
tags: [concurrent]
no-post-nav: true
---

## 类图结构及介绍
AbstractQueuedSynchronizer抽象同步队列简称AQS，它是实现同步器的基础组件，并发包中锁的底层就是使用AQS实现的。

![AQS类图](http://image.wyc1856.club/2019-07-31-14-12-41.png)

> 类
>> AQS继承AbstractOwnableSynchronizer类，独占模式下通过变量exclusiveOwnerThread记录锁的持有线程。

>> Node中的thread变量用来存放进入AQS队列里面的线程；SHARED节点用来标记该线程是获取共享资源时被阻塞挂起后放入AQS队列的，EXCLUSIVE用来标记线程是获取独占资源时被挂起后放入AQS队列的；waitStatus记录当前线程的等待状态，可以为CANCELLED(线程被取消了)、SIGNAL(线程需要被唤醒)、CONDITION(线程在条件队列中等待)、PROPAGATE(释放共享资源时需要通知其他节点)；prev记录当前节点的前驱节点，next记录当前节点的后继节点。

>> ConditionObject条件变量，每一个条件变量对应一个条件队列（单向链表），用来存放调用条件变量的await()方法后被阻塞的线程，可通过调用条件变量的signal()和signalAll()方法来移除条件变量阻塞队列中的线程节点并放入AQS的阻塞队列中，然后激活线程。这块可以类比synchronized代码块和共享变量的wait(),notify()和notifyAll()方法的使用。

> 变量
>> state：AQS中维持的单一状态信息，可以通过getState、setState、compareAndSetState函数修改其值，具体代表的状态信息需子类自己定义。

>> head和tail：分别为AQS内双向阻塞队列的首、尾节点。

>> xxxOffset: 进行CAS操作的变量偏移量。

> 方法
>> 线程同步的关键是对状态值state进行操作。根据state是否属于一个线程，操作state的方式分为独占方式和共享方式。在独占方式下获取和释放资源使用的方法为：void acquire(int arg)、void acquireInterruptibly(int arg)、boolean release(int arg)；在共享方式下获取和释放资源的方法为：void acquireShared(int arg)、void acquireSharedInterruptibly(int arg)、boolean releaseShared(int arg)，其中带interruptibly的方法会对中断请求响应，当调用此类方法的线程请求锁或阻塞时，其他线程中断了此线程，则此线程会抛出InterruptedException异常而返回。

>> boolean tryAcquire(int arg)、boolean tryRelease(int arg)、int tryAcquireShared(int arg)、boolean tryReleaseShared(int arg)、boolean isHeldExclusively()方法需要子类去实现。