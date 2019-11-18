---
layout: post
title: ConcurrentLinkedQueue(线程安全的非阻塞无界队列)
category: note
tags: [concurrent]
no-post-nav: true
---

ConcurrentLinkedQueue内部使用单向链表数据结构来保存队列元素，每个元素被包装成一个Node节点。队列是靠头(head)、尾(tail)节点来维护的，创建队列时头、尾节点指向一个item为null的哨兵节点。head和tail使用volatile修饰来保证内存可见性和有序性，替换头、尾节点的casHead(Node cmp, Node val)、casTail(Node cmp, Node val)方法使用非阻塞CAS算法保证原子性。Node节点中的item和next也是如此。类图结构如下:

![ConcurrentLinkedQueue类图](http://image.wyc1856.club/2019-08-21-15-23-31.png)

## 常用方法及操作
- offer(E e)操作是在队尾添加一个元素，如果传入的参数是null则抛出NPE异常，否则由于ConcurrentLinkedQueue是无界队列，该方法一直会返回true。
- add(E e)操作其实在内部调用的还是offer操作，实现的功能和offer操作一样。
- poll()操作是在队列头部获取并移除一个元素，如果队列为空则返回null。
- peek()操作是获取队列头部一个元素（只获取不移除），如果队列为空则返回null。
- remove(Object o)如果队列里面存在该元素则删除该元素，如果存在多个则删除第一个，并返回true，否则返回false。
- size()操作用来计算队列元素个数，在并发环境下不是很有用，因为CAS没有加锁，所以从调用size方法到返回结果期间有可能增删元素，导致统计的元素个数不精确。
- contains(Object 0)判断队列里面是否含有指定对象，和size方法一样在并发环境下不是很有用。