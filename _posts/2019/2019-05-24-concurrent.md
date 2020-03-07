---
layout: post
title: LinkedBlockingQueue(线程安全的阻塞有界队列)
category: note
tags: [concurrent]
no-post-nav: true
---

LinkedBlockingQueue的内部是通过单向链表实现的，使用头(head)、尾(last)节点来进行出队和入队操作。出队是通过一个ReentrantLock实例(takeLock)来控制同时只能有一个线程可以从队列头部获取元素，入队通过另一个ReentrantLock实例(putLock)来控制同时只能有一个线程可以从队尾插入元素。同时内部还维护了一个初始值为0的原子变量(AtomicInteger)count，用来记录队列元素的个数，同时通过capacity记录队列的容量。另外还包含两个条件变量notEmpty和notFull，notEmpty对应的条件队列用来存放因执行出队(take)操作但队列为空而被阻塞的线程，notFull对应的条件队列用来存放因执行入队(put)操作但队列已满而被阻塞的线程，以此实现了一个生产消费模型。类图结构如下:

![LinkedBlockingQueue类图](http://image.wyc1856.club/2019-08-21-15-22-30.png)

## 常用方法及操作
- offer(E e)操作向队尾插入一个元素，如果队列有空闲则插入成功后返回true，如果队列已满则丢弃当前元素然后返回false。如果e元素为null则抛出NPE异常。通过putLock.lock()获取锁，然后插入e元素到队尾，最后释放锁。
- put(E e)操作和offer操作类似，都是向队尾插入一个元素，但是put操作通过putLock.lockInterruptibly()方法获取锁，表示对中断操作响应。并且当队列已满时会调用notFull.await()阻塞当前线程，加入到notFull对应的条件队列中。
- poll()操作从队列头部获取并移除一个元素，如果队列为空则返回null。
- peek()操作获取队列头部元素但是不从队列里面移除它，如果队列为空则返回null。
- take()操作和poll操作类似，都是获取队头元素并将其从队列中移除。但是take操作通过takeLock.lockInterruptibly()方法获取锁，表示对中断响应。并且当队列为空时会调用notEmpty.await()阻塞当前线程，加入到notEmpty对应的条件队列中。
- remove(Object o)操作用来删除队列中指定的元素，首先进行双重加锁，然后遍历队列直到找到指定的元素，删除该元素并返回true，遍历结束都没找到该元素则返回false，最后释放双重锁，释放锁的顺序和加锁顺序相反。
- size()操作返回当前队列的元素个数，直接返回count.get()即可。

## 小结
![2020-03-06-17-44-58](http://image.wyc1856.club/2020-03-06-17-44-58.png)