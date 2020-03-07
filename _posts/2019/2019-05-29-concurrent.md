---
layout: post
title: ArrayBlockingQueue(线程安全的阻塞有界队列)
category: note
tags: [concurrent]
no-post-nav: true
---

ArrayBlockingQueue 内部是通过数组来实现的。使用 putIndex 来表示入队元素下标，用 takeIndex 来表示出队元素下标，count 变量表示队列中元素的数量。通过 ReentrantLock 类型的 lock 变量加锁来保证入队、出队、删除元素、统计数量时线程安全性。另外和 LinkedBlockingQueue 一样使用 notEmpty、notFull 条件变量来进行出、入队的同步。具体看一下类图结构：
![ArrayBlockingQueue](http://image.wyc1856.club/2020-03-06-19-57-19.png)

## 常用方法及操作
- offer(E e) 入队操作，元素为空则抛出 NPE 异常。首先获取 lock 锁，获取锁失败则加入 AQS 阻塞队列，成功则判断队列是否满，如果未满则加入队尾，修改 putIndex、count + 1、发送 notEmpty.singanl 信号唤醒因调用 take 方法进入 notEmpty 条件阻塞队列的线程。否则丢弃入队元素，返回 false，最后释放锁。
- put(E e) 入队操作，流程和 offer 方法一致，区别在于队列已满的情况下 put 方法会调用 notFull.await() 方法将当前线程加入 notFull 的条件队列。
- poll() 出队操作，首先获取 lock 锁，获取锁失败则加入 AQS 阻塞队列，成功则判断队列是否为空，为空则返回 null，否则队首元素出队，修改 takeIndex、count - 1、发送 notFull.signal 信号唤醒因调用 put 方法进入 notFull 条件阻塞队列的线程，最后释放锁。
- take() 出队操作，流程和 poll 方法一致，区别在于队列为空的情况下 take 方法会调用 notEmpty.await() 方法将当前线程加入 notEmpty 的条件队列。
- peek() 获取队首元素，首先获取锁，返回数组 items 下标为 takeIndex 的元素，如果队列为空则返回 null，最后释放锁。
- remove(E e) 删除指定元素，首先获取锁，然后遍历队列元素，删除遇到的第一个相同的元素并迁移后续队列中的元素，然后返回 true，未找到相同的元素则返回 false，最后释放锁。
- size() 方法，首先获取锁，然后返回 count 的值，最后释放锁。为啥只返回一个 count 的值还要加锁呢，是因为 count 并未通过 volatile 修饰，加锁的目的是保证内存可见性，保证在获取锁后获取到的结果是主存中的数据而非 cpu 缓存或寄存器中的数据。

## 小结
![操作图解](http://image.wyc1856.club/2020-03-06-20-27-10.png)
