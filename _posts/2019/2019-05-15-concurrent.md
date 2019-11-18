---
layout: post
title: JDK常见并发操作类
category: note
tags: [concurrent]
no-post-nav: true
---

## 使用Unsafe类实现CAS操作

Unsafe类为一单例实现，提供静态方法getUnsafe获取Unsafe实例，当且仅当调用getUnsafe方法的类为引导类加载器所加载时才合法，否则抛出SecurityException异常。

### 如何获取实例
其一，从getUnsafe方法的使用限制条件出发，通过Java命令行命令-Xbootclasspath/a把调用Unsafe相关方法的类A所在jar包路径追加到默认的bootstrap路径中，使得A被引导类加载器加载，从而通过Unsafe.getUnsafe方法安全的获取Unsafe实例。
```
java -Xbootclasspath/a: ${path}   // 其中path为调用Unsafe相关方法的类所在jar包路径 
```
其二，通过反射获取单例对象theUnsafe。
```
private static Unsafe reflectGetUnsafe() {
    try {
      Field field = Unsafe.class.getDeclaredField("theUnsafe");
      field.setAccessible(true);
      return (Unsafe) field.get(null);
    } catch (Exception e) {
      log.error(e.getMessage(), e);
      return null;
    }
}
```

## Random及ThreadLocalRandom解析
> 在JDK7之前，Random类作为随机数生成器被广泛使用，但是在多线程环境下，效率较低。因为随机数的生成需要依赖种子，而旧种子生成新种子的算法和通过种子生成随机数的算法是固定的。为了保证多线程环境下生成随机数的随机性，就要保证每一个线程更新种子的操作必须是原子的，Random类通过CAS操作保证了这点，但是当一个线程更新种子时导致其他线程无法更新而自旋，降低了并发效率。

> ThreadLocalRandom被用在高并发场景下来生成随机数，其实现原理类似ThreandLocal类，通过在线程中设置种子(threadLocalRandomSeed)和探针(threadLocalRandomProbe)变量来进行变量的线程隔离。另外，ThreadLocalRandom类的实例instance被声明为static，它里面只包含与线程无关的通用算法，所以它是线程安全的。

## JUC内原子操作类
JUC包内包含AtomicInteger、AtomicLong和AtomicBoolean等原子性操作类，实现原理基本类似，通过volatile实现内存可见性、CAS操作实现原子性来保证线程安全。JDK8中新增了原子性递增递减类LongAdder来克服AtomicLong在高并发场景下CAS操作失败后进行自旋重试CAS白白浪费CPU的问题，通过在内部维护多个Cell元素(一个动态的Cell数组)来分担对单个变量进行争夺的开销。

> LongAdder的结构是啥样的？继承Striped64类，Striped64类维护了三个变量(cells,base,cellsBusy)。cells为Cell数组，默认为null。base是个基础值，默认为0，LongAdder的真实值其实是base的值与cells数组里面所有Cell元素中的value值的累加。cellsBusy状态只有0和1，创建Cell元素，扩容cells数组或者初始化cells数组时，使用CAS操作该变量来保证同时只能有一个线程可以进行相关操作。

> 如何保证线程操作被分配的Cell元素的原子性？Cell类中有一个cas函数通过CAS操作保证线程操作被分配的Cell元素中value值的原子性。另外，Cell类使用@sun.mic.Contended修饰避免伪共享。

> 当前线程应该访问cells数组里面的哪一个Cell元素？首先看cells是否为null，如果为空则当前在基础变量base上进行累加，这个时候就类似AtomicLong的操作。当cells不为空则会通过当前线程的threadLocalRandomProde变量与cells数组元素个数-1进行&运算得到应该访问的Cell元素下标。

> 如何初始化cells数组？一开始cell为null，在基础变量上累加，当CAS操作失败时进行cells数组的初始化。首先通过cellsBusy判断是否有其他线程在初始化cells数组，没有的话初始化cells数组元素个数为2，Cell元素的值目前还是null。

> 线程访问分配的Cell元素有冲突后如何处理？重新计算当前线程的随机值threadLocalRandomProde，以减少下次访问cells元素时的冲突机会。

> Cell数组如何扩容？当cells数组的元素个数小于当前机器的CPU个数并且当前多个线程访问了cells中同一个元素，从而导致冲突使其中一个线程CAS失败时才会进行扩容操作。通过CAS操作设置cellsBusy为1，创建容量为原容量2倍的新数组，并复制Cell元素到扩容后的数组。

## JUC中LockSupport工具类
> 它的主要作用是挂起和唤醒线程。LockSupport类与每一个使用它的线程都会关联一个许可证，在默认情况下线程是不持有许可证的。LockSupport.park()方法在有许可证的情况下直接返回，而没有许可证则会被挂起，直至获取到许可证才会返回，可以通过LockSupport.unPark(Thread thread)给指定线程发放许可证。另外，调用该阻塞线程的interrupt()方法，设置了中断标志或者线程被虚假唤醒park方法也会正常返回。此处的中断操作不同于中断sleep和join方法，不会抛出InterruptedException异常。

## JUC内并发List 
### CopyOnWriteArrayList
> CopyOnWriteArrayList是一个线程安全的ArrayList，对其进行的修改操作都是在底层的一个复制的数组（快照）上进行的，也就是使用了写时复制策略。每一个CopyOnWriteArrayList对象里面有一个array数组对象用来存放具体的元素，ReentrantLock独占锁对象用来保证同时只有一个线程对array进行修改。由于使用了写时复制策略，高并发场景下就会出现弱一致性问题，由CopyOnWriteArrayList创建的迭代器也是弱一致性的，即获取迭代器后，其他线程对原list的增删改对原迭代器是不可见的。