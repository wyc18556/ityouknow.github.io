---
layout: post
title: 并发编程之美笔记
category: node
tags: [concurrent]
no-post-nav: true
---

## 第一部分（基础篇）
### Thread类及Object类相关方法
#### 创建线程
- 继承Thread类，重写run方法；   
- 实现Runnable接口的run方法；
- 实现Callable接口的call方法，然后构建FutureTask实例，再封装成Thread对象，线程start()后通过futureTask.get()获取返回值；

#### 线程的通知与等待
> wait()系列：首先获取共享变量(resource)的监视器锁(synchronized)，调用共享变量上wait()系列方法挂起当前线程，释放监视器锁，进入resource下的线程阻塞队列。

> notify()&notifyAll()：唤醒（进入就绪状态，重新竞争监视器锁及CPU调度）因调用wait()系列方法而阻塞的线程，该方法和wait()系列方法一样，都需要在获取到监视器锁的前提下调用，否则会抛出IllegalMonitorStateException异常。一个需要注意的地方是，在共享变量上调用notifyAll()方法只会唤醒调用这个方法 __之前__ 调用了wait()系列函数而被放入共享变量等待集合里面的线程。

#### 等待线程执行终止
> join()方法，线程A调用了线程B的join方法，线程A会被阻塞，直到线程B执行完毕后线程A才能执行后续的逻辑。注意一点，当线程A阻塞时，其他线程调用线程A的interrupt()方法试图中断线程A时，线程A会抛出InterruptedException异常而返回。

#### 让线程睡眠
> sleep(milliseconds)方法会让当前线程进入睡眠状态（不参与CPU的调度，但是并不会释放所占用的监视器资源）,当睡眠时间到后，该线程会进入就绪状态，重新参与CPU的调度。睡眠期间不能通过其他线程调用本线程的interrupt()方法试图中断线程，本线程会在调用sleep方法的地方抛出InterruptedException异常。

#### 让线程让出CPU
> yield()方法会让当前线程让出CPU使用权，然后处于就绪状态。注意和sleep方法的区别，sleep会使当前线程阻塞挂起指定的时间，这段时间内是不会得到CPU的调度的，但是yield方法只是让出当前剩余的时间片，并没有被阻塞挂起，而是处于就绪状态，线程调度器下一次调度时就有可能调度到当前线程执行。

#### 让线程中断阻塞
> 当线程调用了sleep和join方法后就会进入阻塞状态，阻塞状态下调用interrupt()方法会在调用sleep和join方法的地方抛出InterruptedException异常，可以通过捕获该异常而提前结束阻塞。也可以在调用阻塞方法之前就调用interrupt()方法提前设置中断状态为true，后面调用到阻塞方法时会立即抛出异常而中断阻塞。isInterrupted()和interrupted()方法都是用来判断线程的中断状态的，但还是有区别的，isInterrupted()方法返回的是调用对象的中断状态，而interrupted()返回的是执行该语句的线程的中断状态，并且会清除中断状态。

### 线程死锁
> 四个必要条件：1.资源互斥；2.请求并保持；3.不可剥夺；4.环路等待；

> 避免死锁：只需要破坏掉构成死锁的任一必要条件即可，实际上，目前只有请求并保持和环路等待条件是可以被破坏的。使用资源申请的有序性原则即可避免死锁。

### 守护线程与用户线程
> 守护线程：例如垃圾回收线程；

> 用户线程：例如main线程，JVM进程只有在最后一个非守护线程执行结束后才会结束。线程可以通过setDaemon(boolean)方法设置是否为守护线程。

### ThreadLocal和InheritableThreadLocal
> ThreadLocal：在每一个线程内部都有一个名为threadLocals的成员变量，该变量类型为HashMap，其中key为我们定义的ThreadLocal变量的this引用，value则为我们使用set方法设置的值。具体的介绍可阅读[这篇文章](https://mp.weixin.qq.com/s/K-8aNF3gqg3ekrRbTsjo9w)

> InheritableThreadLocal：我们知道ThreadLocal变量在父线程中设置好后，在子线程中是获取不到的。但是使用InheritableThreadLocal就可以让子线程访问到在父线程中设置的本地变量。同样，在每一个线程内部都有一个名为inheritableThreadLocals的成员变量。创建子线程时会判断父线程的inheritableThreadLocals是否为空，不为空则复制一份到子线程的inheritableThreadLocals中。InheritableThreadLocal继承ThreadLocal，重写了getMap(args)和createMap(args)方法来针对当前线程的inheritableThreadLocals进行操作。

### 并发编程问题
并发编程，为了保证数据的安全，需要满足以下三个特性：
- 原子性：就是在一个操作中cpu不可以在中途暂停然后再调度，既不被中断操作，要么执行完成，要么就不执行。
- 可见性：当多个线程访问同一个变量时，一个线程修改了这个变量的值，其他线程能够立即看得到修改的值。
- 有序性：程序执行的顺序按照代码的先后顺序执行。

> 总结：__缓存一致性问题__ 其实就是 __可见性问题__。而 __处理器优化__ 是可以导致 __原子性问题__ 的。__指令重排__ 会导致 __有序性问题__。

### java内存模型
> [JMM](http://www.hollischuang.com/archives/2550)：一种符合内存模型规范的，屏蔽了各种硬件和操作系统的访问差异的，保证了Java程序在各种平台下对内存的访问效果一致的机制及规范。

> synchronized关键字是一种原子性内置锁。内存语义：进入synchronized代码块（获得锁）会清空锁内本地内存中将会被用到的共享变量，在使用这些共享变量时从主内存中进行加载；退出synchronized代码块（释放锁）时将本地内存中修改的共享变量刷新到主内存中。可以解决共享变量原子性、内存可见性及有序性问题，但是会导致线程上下文切换并带来线程调度开销。

> volatile关键字是一种解决共享变量内存可见性和有序性的非阻塞方案（通过 __内存屏障__ 实现），但是并不能保证原子性。以下两个场景中可以使用volatile来代替synchronized：1、运算结果并不依赖变量的当前值，或者能够确保只有单一的线程会修改变量的值；2、变量不需要与其他状态变量共同参与不变约束。

> 已经有了缓存一致性协议(MESI)，为什么还需要volatile？1、并不是所有的硬件架构都提供了相同的一致性保证，Java作为一门跨平台语言，JVM需要提供一个统一的语义。2、操作系统中的缓存和JVM中线程的本地内存并不是一回事，通常我们可以认为：MESI可以解决缓存层面的可见性问题。使用volatile关键字，可以解决JVM层面的可见性问题。3、缓存可见性问题的延伸：由于传统的MESI协议的执行成本比较大，所以CPU通过Store Buffer和Invalidate Queue组件来解决。但是由于这两个组件的引入，也导致缓存和主存之间的通信并不是实时的。也就是说，MESI协议，可以保证缓存的一致性，但是无法保证实时性。

> CAS(Compare and Swap)是JDK提供的非阻塞原子性操作，它通过硬件保证比较-操作的原子性。JDK中的Unsafe类提供了一系列compareAndSwap*方法。
>> 实现方式：CAS方法对应的有4个操作数，分别为对象内存位置(obj)，对象中的变量的偏移量(valueOffset)，变量预期值(expect)和新的值(update)。其操作含义是，如果对象obj中偏移量为valueOffset的变量的值为expect，则用新值update替换旧值expect。这是处理器提供的一个 __原子性指令__ 。

> 伪共享：缓存行作为CPU高速缓存与主存进行数据交换的单位，当多个线程同时修改同一个缓存行中的多个变量时，由于MESI协议的存在，导致缓存行频繁失效，影响效率。
>> 如何避免伪共享：1、通过字节填充的方式，保证被操作的属性占用的字节数加上前后填充的字节数不小于一个缓存行占有的字节数即可，典型的以空间换时间的操作；2、JDK8之后可以使用注解@sun.misc.Contended来做缓存行的隔离，注意使用此种方案需要添加JVM参数-XX:-RestrictContended。具体可参考[这篇文章](https://blog.csdn.net/qq_27680317/article/details/78486220)

### 锁相关
> 乐观锁与悲观锁：悲观锁认为数据很容易被其他线程修改，所以在数据被处理前先加排他锁，例如在select语句后加for update加行锁。乐观锁认为数据在一般情况下不会造成冲突，所以在访问前不会加排他锁，而是在进行数据提交更新时，才会正式对数据进行校验。例如加版本号标识，在数据被更新时进行版本号校验，更新成功后版本号加一，有点类似CAS操作。

> 公平锁与非公平锁：公平锁标识线程获取锁的顺序是按照线程请求锁的先后时间来决定的，类似队列的思想，非公平锁则不存在这样的限制。ReentrantLock提供了公平锁和非公平锁的实现：
>> 公平锁-ReentrantLock pairLock = new ReentrantLock(true)。

>>非公平锁-ReentrantLock pairLock = new ReentrantLock(false)。如果构造函数不传递参数，则默认为非公平锁。

> 独占锁与共享锁：独占锁保证任何时候都只有一个线程能得到锁，ReentrantLock就是以独占锁的方式实现的。共享锁则可以同时由多个线程持有，例如ReadWriteLock读写锁，它允许一个资源可以被多线程同时进行读操作。独占锁是一种悲观锁，共享锁则是一种乐观锁。

> 可重入锁：当一个线程再次获取它自己已经获得的锁时不会被阻塞，那么我们说该锁是可重入的。synchronized内部维护着计数器monitor和锁的当前持有线程来实现可重入。

> 锁的优化：java中的线程是与操作系统中的线程一一对应的，线程的阻塞挂起和唤醒会导致用户态和内核态的切换，开销很大。synchorized监视器锁在JDK6后优化了很多，当只有一个线程请求锁时锁的状态为 __偏向锁__，有多个线程请求锁但不存在竞争时锁的状态为 __轻量级锁__，存在竞争后也不会立即膨胀为重量级锁，而是会进行 __自旋__ 重试，如果重试一定的次数后还没能获得锁则锁的状态才会升级到 __重量级锁__。还有一点，锁的状态只能升不能降。

## 第二部分(高级篇)

### Random及ThreadLocalRandom解析

> 在JDK7之前，Random类作为随机数生成器被广泛使用，但是在多线程环境下，效率较低。因为随机数的生成需要依赖种子，而旧种子生成新种子的算法和通过种子生成随机数的算法是固定的。为了保证多线程环境下生成随机数的随机性，就要保证每一个线程更新种子的操作必须是原子的，Random类通过CAS操作保证了这点，但是当一个线程更新种子时导致其他线程无法更新而自旋，降低了并发效率。

> ThreadLocalRandom被用在高并发场景下来生成随机数，其实现原理类似ThreandLocal类，通过在线程中设置种子(threadLocalRandomSeed)和探针(threadLocalRandomProbe)变量来进行变量的线程隔离。另外，ThreadLocalRandom类的实例instance被声明为static，它里面只包含与线程无关的通用算法，所以它是线程安全的。

### JUC内原子操作类
JUC包内包含AtomicInteger、AtomicLong和AtomicBoolean等原子性操作类，实现原理基本类似，通过volatile实现内存可见性、CAS操作实现原子性来保证线程安全。JDK8中新增了原子性递增递减类LongAdder来克服AtomicLong在高并发场景下CAS操作失败后进行自旋重试CAS白白浪费CPU的问题，通过在内部维护多个Cell元素(一个动态的Cell数组)来分担对单个变量进行争夺的开销。

> LongAdder的结构是啥样的？继承Striped64类，Striped64类维护了三个变量(cells,base,cellsBusy)。cells为Cell数组，默认为null。base是个基础值，默认为0，LongAdder的真实值其实是base的值与cells数组里面所有Cell元素中的value值的累加。cellsBusy状态只有0和1，创建Cell元素，扩容cells数组或者初始化cells数组时，使用CAS操作该变量来保证同时只能有一个线程可以进行相关操作。

> 如何保证线程操作被分配的Cell元素的原子性？Cell类中有一个cas函数通过CAS操作保证线程操作被分配的Cell元素中value值的原子性。另外，Cell类使用@sun.mic.Contended修饰避免伪共享。

> 当前线程应该访问cells数组里面的哪一个Cell元素？首先看cells是否为null，如果为空则当前在基础变量base上进行累加，这个时候就类似AtomicLong的操作。当cells不为空则会通过当前线程的threadLocalRandomProde变量与cells数组元素个数-1进行&运算得到应该访问的Cell元素下标。

> 如何初始化cells数组？一开始cell为null，在基础变量上累加，当CAS操作失败时进行cells数组的初始化。首先通过cellsBusy判断是否有其他线程在初始化cells数组，没有的话初始化cells数组元素个数为2，Cell元素的值目前还是null。

> 线程访问分配的Cell元素有冲突后如何处理？重新计算当前线程的随机值threadLocalRandomProde，以减少下次访问cells元素时的冲突机会。

> Cell数组如何扩容？当cells数组的元素个数小于当前机器的CPU个数并且当前多个线程访问了cells中同一个元素，从而导致冲突使其中一个线程CAS失败时才会进行扩容操作。通过CAS操作设置cellsBusy为1，创建容量为原容量2倍的新数组，并复制Cell元素到扩容后的数组。

### JUC内并发List 
#### CopyOnWriteArrayList
> CopyOnWriteArrayList是一个线程安全的ArrayList，对其进行的修改操作都是在底层的一个复制的数组（快照）上进行的，也就是使用了写时复制策略。每一个CopyOnWriteArrayList对象里面有一个array数组对象用来存放具体的元素，ReentrantLock独占锁对象用来保证同时只有一个线程对array进行修改。由于使用了写时复制策略，高并发场景下就会出现弱一致性问题，由CopyOnWriteArrayList创建的迭代器也是弱一致性的，即获取迭代器后，其他线程对原list的增删改对原迭代器是不可见的。
 
### JUC中锁原理剖析
#### LockSupport工具类
> 它的主要作用是挂起和唤醒线程。LockSupport类与每一个使用它的线程都会关联一个许可证，在默认情况下线程是不持有许可证的。LockSupport.park()方法在有许可证的情况下直接返回，而没有许可证则会被挂起，直至获取到许可证才会返回，可以通过LockSupport.unPark(Thread thread)给指定线程发放许可证。另外，调用该阻塞线程的interrupt()方法，设置了中断标志或者线程被虚假唤醒park方法也会正常返回。此处的中断操作不同于中断sleep和join方法，不会抛出InterruptedException异常。

### 抽象同步队列(AQS)
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

### 独占锁ReentrantLock的原理
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

### 读写锁ReentrantReadWriteLock的原理
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

#### 写锁的获取与释放(非公平策略)
> void lock()方法
>> 写锁是独占锁，如果当前没有线程获取到读锁和写锁(state=0)，则当前线程通过CAS操作更新state++尝试获取锁，成功则设置exclusiveOwnerThread为当前线程。如果当前已经有线程获取到读锁或写锁(state!=0)，然后判断w=exclusiveCount(state)是否为0，有线程获取了读锁(w==0)或者当前线程非写锁的持有者就挂起当前线程，创建独占节点(Node.EXCLUSIVE)，加入AQS阻塞队列。

> boolean tryLock()方法
>> 尝试获取写锁，和非公平策略的lock()方法类似，未获得锁线程不会阻塞。

> boolean tryLock(long timeout, TimeUnit unit)方法
>> 与tryLock()方法类似，只是多了超时时间设置。

> void unLock()方法
>> 尝试释放锁，state的高16位一定都为0，所以直接操作state--即可，判断state是否为0，为0则释放锁(设置exclusiveOwnerThread为null)。

#### 读锁的获取与释放(非公平策略)
> void lock()方法
>> 首先判断当前是否有其他线程获取了写锁(exclusiveCount(state) != 0 && getExclusiveOwnerThread() != current)，直接返回-1。然后调用AQS的doAccquired方法把当前线程放入AQS阻塞队列。如果当前线程已经获取了写锁，则也能继续获取读锁。但是要注意，读锁处理事情完毕后，要记得把读锁和写锁都释放掉，不能只释放写锁（没明白为啥要这样做）。如果当前没有线程获取写锁(exclusiveCount(state) == 0)，则计算出读锁计数(r = sharedCount(state))，然后尝试获取锁(逻辑比较复杂，暂时先这样吧)。

> boolean tryLock()
>> 尝试获取读锁，逻辑和lock()类似，区别是未获取到锁线程并不会阻塞。

> boolean tryLock(long timeout, TimeUnit unit)方法
>> 与tryLock()方法类似，多了超时时间设置，对线程中断操作响应。

> void unlock()
>> 尝试释放读锁，循环尝试通过CAS操作state高16位减一，CAS操作成功则判断当前state是否为0，为0则说明当前无读线程占用读锁。tryReleaseShared方法返回true，然后调用doReleaseShared方法释放一个由于获取写锁而被阻塞的线程。

### ConcurrentLinkedQueue(线程安全的非阻塞无界队列)
#### 类图结构
![ConcurrentLinkedQueue类图](http://image.wyc1856.club/2019-08-21-15-23-31.png)
底层使用单向链表数据结构来保存队列元素，每个元素被包装成一个Node节点。队列是靠头(head)、尾(tail)节点来维护的，创建队列时头、尾节点指向一个item为null的哨兵节点。head和tail使用volatile修饰来保证内存可见性和有序性，替换头、尾节点的casHead(Node cmp, Node val)、casTail(Node cmp, Node val)方法使用非阻塞CAS算法保证原子性。Node节点中的item和next也是如此。

#### 常用方法及操作
- offer(E e)操作是在队尾添加一个元素，如果传入的参数是null则抛出NPE异常，否则由于ConcurrentLinkedQueue是无界队列，该方法一直会返回true。
- add(E e)操作其实在内部调用的还是offer操作，实现的功能和offer操作一样。
- poll()操作是在队列头部获取并移除一个元素，如果队列为空则返回null。
- peek()操作是获取队列头部一个元素（只获取不移除），如果队列为空则返回null。
- remove(Object o)如果队列里面存在该元素则删除该元素，如果存在多个则删除第一个，并返回true，否则返回false。
- size()操作用来计算队列元素个数，在并发环境下不是很有用，因为CAS没有加锁，所以从调用size方法到返回结果期间有可能增删元素，导致统计的元素个数不精确。
- contains(Object 0)判断队列里面是否含有指定对象，和size方法一样在并发环境下不是很有用。

### LinkedBlockingQueue(线程安全的阻塞有界队列-链表方式实现)
#### 类图结构
![LinkedBlockingQueue类图](http://image.wyc1856.club/2019-08-21-15-22-30.png)
LinkedBlockingQueue的内部是通过单向链表实现的，使用头(head)、尾(last)节点来进行出队和入队操作。出队是通过一个ReentrantLock实例(takeLock)来控制同时只能有一个线程可以从队列头部获取元素，入队通过另一个ReentrantLock实例(putLock)来控制同时只能有一个线程可以从队尾插入元素。同时内部还维护了一个初始值为0的原子变量(AtomicInteger)count，用来记录队列元素的个数，同时通过capacity记录队列的容量。另外还包含两个条件变量notEmpty和notFull，notEmpty对应的条件队列用来存放因执行出队(take)操作但队列为空而被阻塞的线程，notFull对应的条件队列用来存放因执行入队(put)操作但队列已满而被阻塞的线程，以此实现了一个生产消费模型。

#### 常用方法及操作
- offer(E e)操作向队尾插入一个元素，如果队列有空闲则插入成功后返回true，如果队列已满则丢弃当前元素然后返回false。如果e元素为null则抛出NPE异常。通过putLock.lock()获取锁，然后插入e元素到队尾，最后释放锁。
- put(E e)操作和offer操作类似，都是向队尾插入一个元素，但是put操作通过putLock.lockInterruptibly()方法获取锁，表示对中断操作响应。并且当队列已满时会调用notFull.await()阻塞当前线程，加入到notFull对应的条件队列中。
- poll()操作从队列头部获取并移除一个元素，如果队列为空则返回null。
- peek()操作获取队列头部元素但是不从队列里面移除它，如果队列为空则返回null。
- take()操作和poll操作类似，都是获取队头元素并将其从队列中移除。但是take操作通过takeLock.lockInterruptibly()方法获取锁，表示对中断响应。并且当队列为空时会调用notEmpty.await()阻塞当前线程，加入到notEmpty对应的条件队列中。
- remove(Object o)操作用来删除队列中指定的元素，首先进行双重加锁，然后遍历队列直到找到指定的元素，删除该元素并返回true，遍历结束都没找到该元素则返回false，最后释放双重锁，释放锁的顺序和加锁顺序相反。
- size()操作返回当前队列的元素个数，直接返回count.get()即可。