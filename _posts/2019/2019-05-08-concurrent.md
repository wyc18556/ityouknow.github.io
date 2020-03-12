---
layout: post
title: 线程类常见操作
category: note
tags: [concurrent]
no-post-nav: true
---

## Thread类及Object类相关方法

### 创建线程
- 继承Thread类，重写run方法；   
- 实现Runnable接口的run方法；
- 实现Callable接口的call方法，然后构建FutureTask实例，再封装成Thread对象，线程start()后通过futureTask.get()获取返回值；

### Thead 类 run 方法和 start 方法的区别
start 方法是启动一个新的线程，执行完后线程处于就绪状态，等待 cpu 调度才会在新线程内执行 run 方法的逻辑。而直接调用 run 方法就是相当于普通调用 thread 对象的 run 方法，还是在当前线程内执行的

### 线程的通知与等待
> wait()系列：首先获取共享变量(resource)的监视器锁(synchronized)，调用共享变量上wait()系列方法挂起当前线程，释放监视器锁，进入resource下的线程阻塞队列。阻塞挂起期间如果其他线程调用了本线程的interrput()方法试图中断阻塞时，当前线程会在调用resource.wait()方法的地方抛出InterruptedException异常而返回。

> notify()&notifyAll()：唤醒（进入就绪状态，重新竞争监视器锁及CPU调度）因调用wait()系列方法而阻塞的线程，该方法和wait()系列方法一样，都需要在获取到监视器锁的前提下调用，否则会抛出IllegalMonitorStateException异常。一个需要注意的地方是，在共享变量上调用notifyAll()方法只会唤醒调用这个方法 __之前__ 调用了wait()系列函数而被放入共享变量等待集合里面的线程。

### 等待线程执行终止
> join()方法，线程A调用了线程B的join方法，线程A会被阻塞，直到线程B执行完毕后线程A才能执行后续的逻辑。注意一点，当线程A阻塞时，其他线程调用线程A的interrupt()方法试图中断线程A时，线程A会在调用线程B的join方法处抛出InterruptedException异常而返回。

### 让线程睡眠
> sleep(milliseconds)方法会让当前线程进入睡眠状态（不参与CPU的调度，但是并不会释放所占用的监视器资源）,当睡眠时间到后，该线程会进入就绪状态，重新参与CPU的调度。睡眠期间不能通过其他线程调用本线程的interrupt()方法试图中断线程，本线程会在调用sleep方法的地方抛出InterruptedException异常。

### 让线程让出CPU
> yield()方法会让当前线程让出CPU使用权，然后处于就绪状态。注意和sleep方法的区别，sleep会使当前线程阻塞挂起指定的时间，这段时间内是不会得到CPU的调度的，但是yield方法只是让出当前剩余的时间片，并没有被阻塞挂起，而是处于就绪状态，线程调度器下一次调度时就有可能调度到当前线程执行。

### 让线程中断阻塞
> 当线程调用了sleep和join方法后就会进入阻塞状态，阻塞状态下调用interrupt()方法会在调用sleep和join方法的地方抛出InterruptedException异常，可以通过捕获该异常而提前结束阻塞。也可以在调用阻塞方法之前就调用interrupt()方法提前设置中断状态为true，后面调用到阻塞方法时会立即抛出异常而中断阻塞。isInterrupted()和interrupted()方法都是用来判断线程的中断状态的，但还是有区别的，isInterrupted()方法返回的是调用对象的中断状态，而interrupted()返回的是执行该语句的线程的中断状态，并且会清除中断状态。

## 线程死锁
> 四个必要条件：1.资源互斥；2.请求并保持；3.不可剥夺；4.环路等待；

> 避免死锁：只需要破坏掉构成死锁的任一必要条件即可，实际上，目前只有请求并保持和环路等待条件是可以被破坏的。使用资源申请的有序性原则即可避免死锁。

## 守护线程与用户线程
> 守护线程：例如垃圾回收线程；

> 用户线程：例如main线程，JVM进程只有在最后一个非守护线程执行结束后才会结束。线程可以通过setDaemon(boolean)方法设置是否为守护线程。

## ThreadLocal和InheritableThreadLocal
> ThreadLocal：在每一个线程内部都有一个名为threadLocals的成员变量，该变量类型为ThreadLocalMap，内部维护了一个Entry数组，默认容量为16。通过ThreadLocal的threadLocalHashCode定位数组下表，其中Entry的value就是threadLocal.set方法里设置的值。具体的介绍可阅读[这篇文章](https://mp.weixin.qq.com/s/K-8aNF3gqg3ekrRbTsjo9w)

> InheritableThreadLocal：我们知道ThreadLocal变量在父线程中设置好后，在子线程中是获取不到的。但是使用InheritableThreadLocal就可以让子线程访问到在父线程中设置的本地变量。同样，在每一个线程内部都有一个名为inheritableThreadLocals的成员变量。创建子线程时会判断父线程的inheritableThreadLocals是否为空，不为空则复制一份到子线程的inheritableThreadLocals中。InheritableThreadLocal继承ThreadLocal，重写了getMap(args)和createMap(args)方法来针对当前线程的inheritableThreadLocals进行操作。