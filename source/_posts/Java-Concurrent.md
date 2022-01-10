---
title: Java-Concurrent
date: 2022-01-08 15:45:18
tags: Java
cover: /images/JavaLogo.png
---

# Java高并发学习

[TOC]

## 基本概念

并发：多个任务交替执行

并行：同时执行

临界区：一种公共资源，但是同一时间只能被一个线程访问。

饥饿：线程缺少资源，一直无法执行。

活锁：互相谦让，但还是不能满足执行条件。

### 并发级别

阻塞：得到锁之前，无法执行。

无饥饿：

1. 公平锁

   不会出现饥饿。

2. 非公平锁

   可能出现饥饿。

3. 无障碍

   乐观，并发程度低

   线程之间不会因为临界区的问题被挂起，但检测到冲突就回滚。

4. 无锁

   所有的线程都能访问临界区。cas

### JMM Java内存模型

并发三大特性：

1. 原子性

   一个最小的操作，不会被其他线程干扰。

2. 可见性

   当一个线程修改了共享变量的值，其他线程立即可见。如果不使用volatile，变量则暂存在Java工作内存中，不会立即刷新主存。

3. 有序性

   总结为：在本线程内观察，所有的操作都是有序的；但在一个线程中观察另一个线程，所有的操作都是无序的。

   无序主要体现在**指令重排序**和**工作内存和主存同步延迟**现象。

volatile关键字保证了有序性，可见性。

synchronized保证了全部特性。

## 线程的基本操作

1. 新建
2. 终止 stop
3. 中断 interrupt
4. 等待 wait 和通知 notify
5. （废弃）挂起 suspend和继续执行 resume
6. 等待线程结束 join 和让出cpu yeild

### 线程的状态

1. 初始(NEW)：新创建了一个线程对象，但还没有调用start()方法。
2. 运行(RUNNABLE)：可运行的，由CPU进行调度运行。
3. 阻塞(BLOCKED)：synchronized请求锁monitor阻塞。
4. 等待(WAITING)：object.wati()阻塞等待，需要object.notify()来唤醒。IO。
5. 超时等待(TIMED_WAITING)：Thread.Sleep()睡眠，超时唤醒，或中断唤醒。
6. 终止(TERMINATED)：表示该线程已经执行完毕。

## 驻守后台-守护线程Deamon

线程有两种：守护线程，用户线程

正如名字一样，是系统的守护者，在后台守护用户线程。

如果所有用户线程都结束了，那么守护线程会默默退出。

默认创建的是用户线程，不会因main线程的结束而退出。

```java
public class TestDeamonThread {
    public static void main(String[] args) {
        Thread t = new Thread(new MyRunnable());
        t.setDaemon(true); // 设置守护线程后，主线程结束，守护线程默默退出
        t.start();
    }
}
```

## Java并发工具

### Synchronized

- 同步类
- 同步对象
- 同步方法
- 同步静态方法/相当于同步类
- 同步静态变量/相当于同步类

### Object / 属于管程的实现

只能与Synchronized配合使用

- obj.wait(); 释放锁monitor，阻塞等待notify和获得锁。
- obj.notify(); 唤醒随机一个因wait()阻塞的对象。
- obj.notifyAll(); 唤醒所有因wait()阻塞的对象。

## JUT 同步工具

### 可重入锁ReentrantLock

可以反复进入的锁

```Java
lock.lock();
lock.lock();
lock.unlock();
lock.unlock();
```

#### 方法：

1. lock(); 锁
2. tryLock(); 请求锁，失败返回false
3. tryLock(time, unit) 请求锁，time unit后超时，返回false
4. lockInterruptibly(); 可中断锁，请求锁时优先响应中断，中断后会放弃请求，并释放已有的锁。
5. unlock() ; 释放锁

### 可重入锁的好搭档Condition/管程实现

#### 方法

与Object的类似。

- await();
- signal();
- signalAll();

### 倒计数器CountDownLatch

倒数计数器latch，数到0释放。

#### 方法

- await();阻塞等待count为0
- countDown()，count--;

### 循环屏障CyclicBarrier

把线程阻止到屏障外。

#### 方法

- await(); 当n个线程都达到屏障后，各个线程才会继续运行。

### 信号量Semaphore

表示可进入临界区/可获得资源的线程数量。

达到0时，阻塞请求的线程。

#### 方法

- acquire(): 获取，数量-1
- release(): 释放，数量+1

## 线程池

为了避免系统频繁的创建和销毁线程浪费资源，让创建的线程复用。用空间换时间。

创建线程==》从线程池中获取活跃线程，销毁线程==》把线程还回去。

#### 线程池如何重用？

```java
// 线程池核心代码：
static final class RunnableAdapter<T> implements Callable<T> {
  final Runnable task;
  final T result;
  RunnableAdapter(Runnable task, T result) {
    this.task = task;
    this.result = result;
  }
  public T call() {
    task.run(); // *调用run()方法
    return result;
  }
}
```



#### 创建多少线程？

不需要太精确，但是不能过大或过小。需要考虑CPU数量，内存大小等因素。

一般来说，公式：threads = Ncpu * Ucpu * (1 + W / C)

**公式解读：** 重点在于区分任务是IO密集型（W）还是计算密集型（C）：如果是IO密集型，则W很大，所以应该开更多线程；如果是计算密集型，则C很大，所以开更多的线程也没用。

- Ncpu：Cpu数量
- Ucpu：期望的Cpu使用率
- W / C： 等待时间与计算时间的比值

### JDK的Executor框架

Executors是一个线程工厂。通过Executors可以取得一个拥有特定功能的线程池。

#### 工厂方法

```java
public static ExecutorService newFixedThreadPool(int nThreads)
public static ExecutorService newSingleThreadExecutor()
public static ExecutorService newCachedThreadPool()
public static ScheduledExecutorService newSingleThreadScheduledExecutor()
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize)
```

- newFixedThreadPool() ：返回一个固定数量的线程池。有任务提交时，若线程池有空闲线程，立即执行。若无，任务放在队列中，空闲时执行。
- newSingleThreadExecutor()：只有一个线程的线程池。也有任务队列。
- newCachedThreadPool()：一个可根据实际情况调整线程数量的线程池。如果全部线程都在工作，有新任务提交时，就创建新线程处理任务。
- newSingleThreadScheduledExecutor()：线程池大小为1，扩展了延时任务功能。
- newScheduledThreadPool()：同上，但是可以指定线程池数量

#### 常用方法

- execute()：无返回值的提交
- submit()：有返回值的提交（异步的概念）
- shutdown()：不允许提交任务，会立即返回。可以用awaitTermination等待全部任务执行完毕。
- shutdownNow()：会调用每个线程的interrupt。
- isTeminated()：是否执行完毕

#### 计划任务

```java
// 这两类属于计划任务
public static ScheduledExecutorService newSingleThreadScheduledExecutor()
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize)
  // 测试
ScheduledExecutorService executor = Executors.newScheduledThreadPool(10);
```

计划任务的三个方法：

```java
// 在给定时间对任务进行调度
public ScheduledFuture<?> schedule(Runnable command,
                                       long delay, TimeUnit unit);
// 周期性调度，以开始时间为起点，固定频率。
public ScheduledFuture<?> scheduleAtFixedRate(Runnable command,
                                                  long initialDelay,
                                                  long period,
                                                  TimeUnit unit);
// 周期性调度，以上一个任务结束时间为起点，固定频率。
public ScheduledFuture<?> scheduleWithFixedDelay(Runnable command,
                                                     long initialDelay,
                                                     long delay,
                                                     TimeUnit unit);
```

#### 计划任务特例

执行时间超过调度时间会发生什么？

- scheduleAtFixedRate(): 立即执行
- scheduleWithFixedDelay()：等待调度时间才执行

### 线程池内部实现 ThreadPoolExecutor

```Java
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue) {
    this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
         Executors.defaultThreadFactory(), defaultHandler);
}
```

参数解析：

1. corePoolSize 线程数量
2. maximumPoolSize 最大线程数量
3. keepAliveTime 当**线程数量**超过**corePoolSize数量**，在多长时间后会被摧毁
4. unit **keepAliveTime**的时间单位
5. **workQueue** 任务队列，被提交但尚未执行的任务，使用**BlockingQueue**接口对象
6. threadFactory 用于创建线程的工厂
7. **handler** 拒绝策略。任务太多来不及处理，如何拒绝任务

#### 核心参数之间的关系

添加线程：

​	小于corePoolSize：分配执行

​	大于corePoolSize：加入任务队列，等待执行

加入任务队列：

​	看具体任务队列处理办法。

任务队列满了，提交线程池：

​	创建新线程执行，如果线程总数大于Max，执行拒绝策略。

#### 线程池何时创建线程？

- 线程池创建后，无任何线程，此时每提交一个任务，就会根据线程工厂创建一个线程，直到线程数量等于核心线程数量，提交任务队列处理。
- 任务队列提交失败后，会创建线程或执行拒绝策略。

#### 任务队列 workQueue

- 直接提交的队列：使用**SynchronousQueue**，新的任务直接提交给线程池。
- 有界的任务队列：使用**ArrayBlockingQueue**，如果超过队列大小，提交线程池。
- 无界的任务队列：使用**LinkedBlockingQueue**，除非系统资源耗尽，否则无限入队。
- 优先任务队列：使用**PriorityBlockingQueue**，可以控制任务的先后执行顺序，是一个特殊的无界队列。

#### 任务队列对应实现

- newFixedThreadPool(int nThreads)

  使用**无界任务队列LinkedBlockingQueue**，corePoolSize可以和maximumPoolSize相等。当线程数到达max时且任务提交非常频繁，任务队列迅速膨胀耗尽资源。

- newSingleThreadExecutor()

  同上，不过max大小为1。

- newCachedThreadPool()

  使用**直接提交队列SynchronousQueue**，corePoolSize为0，为了能直接提交到任务队列，put任务后会立即被take去创建新线程执行。60秒内回收新线程。

  同样，也可能因为大量任务耗尽资源。

#### 拒绝策略 handler

应对超负载。拒绝策略如下：

- **AbortPolicy策略**：直接抛异常，阻止系统正常工作。
- **CallerRunsPolicy**：在调用者栈中运行当前被丢弃的任务。
- **DiscardOldestPolicy策略**：丢弃最老的一个任务，并再次提交该任务。
- **DiscardPolicy**：默默丢弃无法处理的任务。

#### 自定义线程创建：ThreadFactory

线程池是为了线程复用，那么开始的线程从哪来？ThreadFactory

主要用处：设置线程名字、组、优先级、守护线程等信息。

## JDK并发容器

jdk提供了一系列线程安全的并发容器。

#### 高并发容器

- ConcurrentHashMap : 线程安全的HashMap
- CopyOnWriteArrayList: 线程安全的ArrayList，适合读多写少。
- ConcurrentLinkedListQueue：线程安全的LinkedList
- BlockingQueue：ArrayBlockingQueue阻塞队列。
- ConcurrentSkipListMap: 线程安全跳表。

#### 普通线程安全容器

- HashTable

- Vector

- Collections.synchronizedXXX(List/Map)包装的容器

  此方法与HashTable、Vector的线程安全原理相似，都是一律使用Synchronized关键字来实现。

### 阻塞队列 BlockingQueue

阻塞队列接口有许多不同的实现。

- **ArrayBlockingQueue**

  数组实现的阻塞队列。不能扩容。

- **SynchronousQueue**

  一个特殊的阻塞队列，其实并不存放任何数据。put操作要等待take操作，take操作要等待put操作。

- **LinkedBlockingQueue**

  链表实现的阻塞队列。

- **PriorityBlockingQueue**

  带优先级的阻塞队列

## 锁的优化

1. 减少锁持有时间

   只有在必要时进行同步。

2. 减少锁粒度

   分段锁，如ConcurrentHashMap被分为16个段

   运气好的情况下，可以并发16个线程。

   缺点，系统需要全局锁时，消耗增大。如需要取得size()，就要取得所有是分段锁。

3. 用读写分离锁来替换独占锁

   写的时候不能写，读的时候可以读写。如CopyOnWriteArrayList

4. 锁分离

   如LinkedBlockingQueue中，take()操作和put()操作可以并发。只有take()和take()/put()和put()之间存在竞争，所以可以把take()和put()分离。

5. 锁粗化

   如果对同一个锁不停地请求，同步和释放。会浪费宝贵的资源，不利于性能优化。

   ```java
   for (int i = 0; i < n; i++) {
     synchronized(lock) {
       // do sth.
     }
   }
   ```

   应该改成

   ```java
   synchronized(lock) {  for (int i = 0; i < n; i++) {      // do sth.  } }
   ```

   把synchronized，锁粗化。

### Java虚拟机锁优化

#### 偏向锁

核心思想：如果一个线程获得一个锁，那么再次请求时，如果锁没有被其他线程获取，就不需要任何同步操作。

- 原理：尝试使用CAS把获取到这个锁的线程ID记录到**锁对象头的标记字段**中，如果锁没有被其他线程获取，那么获取过的线程再次请求锁时，不需要进行任何同步操作。
- 锁对象头的标记字段：偏向线程ID，偏向时间戳，GC年龄--01

#### 轻量级锁

如果偏向锁失败，虚拟机不会立即挂起线程，会尝试轻量级锁。

- 原理：尝试用CAS将**锁对象头的标记字段**指向一个线程栈中的**锁记录空间**

  如果操作成功，那么该线程获得了锁。

- 锁对象头的标记字段：指向锁记录的指针--00

如果有，顺利进入临界区，否则说明其他线程获得了锁。膨胀为重量级锁。

#### 自旋锁

锁膨胀后，为了避免线程真实地挂起，虚拟机还会做一次努力-自旋。

虚拟会会让当前线程做几个空循环（默认10），循环后如果可以得到锁，那么就进入临界区，否则挂起。

#### 重量级锁

synchronized通过Monitor来实现线程同步，Monitor是依赖于底层的操作系统的Mutex Lock（互斥锁）来实现的线程同步。

Mutex Lock也是通过CAS和挂起等待的操作来实现的，不过这部分由内核处理，Java的锁大部分在用户态实现。

#### 锁消除

Java虚拟机在JIT编译时，会除去不可能存在共享资源竞争的锁。

根据`逃逸分析`技术，判断一个变量是否会逃出某一个作用域。

如果不会，那就不必要加锁。

#### ThreadLocal

ThreadLocal:针对某一个线程的全局变量

stacic:全部线程的全局变量

#### 最佳实践

多线程环境中。有一个处理流程，每个流程都需要操作同一个上下文。

最简单的做法是:doSth1(x,x,x,ctx),doSth2(x,x,x,ctx),doSth3(x,x,x,ctx)... 使用参数传递上下文。

错误的做法是：使用static ctx来使ctx变成全局变量。

正确的做法：使用LocalMap使ctx线程隔离。

#### 引用分析

![截屏2020-03-10下午4.52.01](../../../Notes/Java/assets/%25E6%2588%25AA%25E5%25B1%258F2020-03-10%25E4%25B8%258B%25E5%258D%25884.52.01.png)

#### Thread原理

`ThreadLocal`的实现是这样的：每个`Thread` 维护一个 `ThreadLocalMap` ，这个map的 `key` 是 `ThreadLocal`实例本身，`value` 是真正需要存储的对象。

也就是说 `ThreadLocal` 本身并不存储值，它只是作为一个 `key` 来让线程从 `ThreadLocalMap` 获取 `value`。值得注意的是图中的虚线，表示 `ThreadLocalMap` 是使用 `ThreadLocal` 的弱引用作为 `Key` 的，弱引用的对象在 GC 时会被回收。

#### Thread内存泄漏

`ThreadLocalMap`使用`ThreadLocal`的弱引用作为`key`，如果一个`ThreadLocal`没有外部强引用来引用它，那么系统 GC 的时候，这个`ThreadLocal`势必会被回收，这样一来，`ThreadLocalMap`中就会出现`key`为`null`的`Entry`，就没有办法访问这些`key`为`null`的`Entry`的`value`，如果当前线程再迟迟不结束的话，这些`key`为`null`的`Entry`的`value`就会一直存在一条强引用链：`Thread Ref -> Thread -> ThreaLocalMap -> Entry -> value`永远无法回收，造成内存泄漏。

其实，`ThreadLocalMap`的设计中已经考虑到这种情况，也加上了一些防护措施：在`ThreadLocal`的`get()`,`set()`,`remove()`的时候都会清除线程`ThreadLocalMap`里所有`key`为`null`的`value`。

#### 源码分析

set方法：

```java
// threadLocal的set方法public void set(T value) {  Thread t = Thread.currentThread(); // 获取当前线程  ThreadLocalMap map = getMap(t); // 从当前线程里获取map(这个map是线程私有的)  if (map != null)    map.set(this, value); // 关键点，调用线程私有的map的set方法  else    createMap(t, value);}// 线程私有的map的set方法的关键步骤int i = key.threadLocalHashCode & (len-1); // 除留余数法// 发生hash冲突（原因：一个线程中的ThreadLocal变量太多）// 开地址法，和HashMap相比，HashMap采用的是链地址法for (Entry e = tab[i];     e != null;     e = tab[i = nextIndex(i, len)]) {  ThreadLocal<?> k = e.get();  if (k == key) {    e.value = value;    return;  }  if (k == null) {    replaceStaleEntry(key, value, i);    return;  }}// 最关键的赋值步骤：tab[i] = new Entry(key, value); // key为threadLocal，value是自定义值
```

#### 常见使用

```java
ThreadLocal<SimpleDateFormat> t = new ThreadLocal<>();
```

因为SimpleDateFormat.parse()方法不是线程安全的。

因为SimpleDateFormat中的Calendar是线程不安全的。

多个线程之间共享变量calendar，并修改calendar。因此在多线程环境下，当多个线程同时使用相同的SimpleDateFormat对象（如static修饰）的话，如调用format方法时，多个线程会同时调用calender.setTime方法，导致time被别的线程修改，因此线程是不安全的。

#### ThreadLocal原理与实现

每个 Thread 都有一个 ThreadLocal.ThreadLocalMap 对象。

当调用一个 ThreadLocal 的set方法时，先得到当前线程的 ThreadLocalMap 对象，然后将 ThreadLocal->value 键值对插入到该 Map 中。

#### ThreadLocal 业务

...

## 无锁

### CAS Compare And Set

原子操作。

比较然后设置。

三个参数:

- 变量V
- 预期值E
- 新值N

仅当V等于E时，把V更新为N。否则失败。

这样就保证了只有一个线程可以操作成功。

### 无锁的线程安全整数: AtomicInteger

核心字段：value，valueOffset

- value 保存值
- valueOffset 保存偏移量，变量在内存中的地址。

通过死循环CAS实现线程安全。

### 无锁的对象引用:AtomicReference



### 带有时间戳的对象引用：AtomicStampedReference

解决了CAS中不能判断ABA的问题。

加入时间戳参数，只有时间戳也匹配时才更改新值。

### 数组也能无锁：AtomicIntegerArray

## 乐观锁与悲观锁

#### 悲观锁

- synchronized
- ReentrantLock

#### 乐观锁

- CAS实现
- 版本号机制实现（MVCC）

## synchronized的使用细节

- 修饰方法：锁实例
- 修饰静态方法：锁类
- 修饰对象：锁对象

## Syn和Lock实现原理

### 区别

- 使用synchronized，JVM会有相应的锁优化，（偏向，轻量级，自旋）
- lock可以是公平锁，synchronized不是公平锁
- lock有许多好用的特性：trylock、可中断，公平，配合Condition使用。

### 相同点

- 可重入

### 1.synchronized(syn)原理

- 被synchronized修饰的代码块，编译的字节码中会被monitorenter和monitorexit包围。
- 执行monitorenter时，首先要尝试获取对象的锁。如果获得成功，把锁的计数器加1。相应的，monitorexit会把锁的计数器减1.

### 2. ReentrantLock原理

[美团AQS分析](https://tech.meituan.com/2019/12/05/aqs-theory-and-apply.html)仅供参考，内容有误（非公平锁图解出错）

[JUC必知必会](https://juejin.im/post/5e4f3cede51d4526c80e99cf)

- 通过AQS实现
- 双向链表实现的同步队列
- Node的waitStatus
- state操作

### AQS AbstractQueuedSynchronizer

同步队列，JUC工具几乎都是通过**组合AQS**，重写某些方法实现的。

#### AQS核心思想

1. 用CAS尝试改变state，如果成功则获得锁
2. CAS失败，就加入同步队列，通过同步机制来有序请求锁。

#### 同步机制



### 锁的维度

- 是否公平

### AQS的三个维度

- 是否**可中断**
- 是否**超时**
- 是否**共享**

#### AQS是否可共享：共享模式shareMode，独占模式exclusiveMode

```java
private volatile int state; // 同步状态state
```

独占模式：state初始为0, state为0时，CAS操作+1，成功则获得锁。

共享模式：state初始为n，每当一个线程获得锁-1，当state为0时阻塞。

#### state操作

- getState
- setState
- compareAndSetState

#### 源码解读

尝试获取锁：lock

```java
final void lock() {  // CAS操作state  if (compareAndSetState(0, 1))    // CAS成功，设置独占锁    setExclusiveOwnerThread(Thread.currentThread());   else    // CAS设置state失败，加入同步队列    acquire(1); // 模板方法}
```

```java
// acquire模板public final void acquire(int arg) {    if (!tryAcquire(arg) // 再次尝试获取，标记1        &&        acquireQueued(addWaiter(Node.EXCLUSIVE), arg)) // 加入队列, 标记2        selfInterrupt();}
```

1. 再次尝试获取锁，tryAcquire标记1导读：

```java
// tryAcquire 标记1protected final boolean tryAcquire(int acquires) {  return nonfairTryAcquire(acquires); // 标记1}// 非公平锁的尝试获取//  nonfairTryAcquire 标记1final boolean nonfairTryAcquire(int acquires) {  final Thread current = Thread.currentThread();  int c = getState();  // 如果state为0，再尝试一次CAS获取锁  if (c == 0) {    if (compareAndSetState(0, acquires)) {      setExclusiveOwnerThread(current);      return true; // CAS成功，获得锁    }  }  // 如果自己是当前锁的持有者，重入操作，state++  else if (current == getExclusiveOwnerThread()) {    int nextc = c + acquires;    if (nextc < 0) // overflow      throw new Error("Maximum lock count exceeded");    setState(nextc);    return true;  }  return false; // CAS失败，自己也不是持有者，失败}
```

2. 加入同步队列，标记2导读

添加到队尾：addWaiter：

```java
// 主要思想：CAS操作添加节点到同步队列队尾private Node addWaiter(Node mode) {  Node node = new Node(Thread.currentThread(), mode);  Node pred = tail;  if (pred != null) {    node.prev = pred;    // CAS    if (compareAndSetTail(pred, node)) {      pred.next = node;      return node;    }  }  enq(node);  return node;}
```

2. 在同步队列中自旋请求锁，标记2导读

   在队列中自旋请求锁：acquireQueued：

```java
final boolean acquireQueued(final Node node, int arg) {  boolean failed = true;  try {    boolean interrupted = false;    // 自旋操作，要么获得锁，要么阻塞（防止自旋浪费资源）    for (;;) {      // 获取当前node的前驱      final Node p = node.predecessor();      // 1.如果自己在队列头部，尝试CAS请求锁（和之前一样）      if (p == head && tryAcquire(arg)) {        // 请求锁成功，设置自己为队头        setHead(node);         p.next = null; // help GC，删除自己（因为已经获得锁了）        failed = false;        return interrupted; // 返回      }      // 2. 判断是否需要阻塞（根据waitStatus）      if (shouldParkAfterFailedAcquire(p, node) &&          parkAndCheckInterrupt())        interrupted = true;    }  } finally {    if (failed)      cancelAcquire(node);  }}
```

`shouldParkAfterFailedAcquire(p, node) && parkAndCheckInterrupt())`分析：

```java
// 靠前驱节点判断是否需要阻塞private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {  int ws = pred.waitStatus; // 前驱节点状态  // 前驱处于唤醒状态  if (ws == Node.SIGNAL)    return true;  // 前驱处于取消状态  if (ws > 0) {    // 把 队列从自己往队头遍历，删除处于取消状态的节点    do {      node.prev = pred = pred.prev;    } while (pred.waitStatus > 0);    pred.next = node;  } else {    // 设置前驱唤醒    compareAndSetWaitStatus(pred, ws, Node.SIGNAL);  }  return false;}
```

```java
// 阻塞当前线程private final boolean parkAndCheckInterrupt() {    LockSupport.park(this);    return Thread.interrupted();}
```

#### waitStatus枚举表

| 枚举      | 含义                                                         |
| :-------- | :----------------------------------------------------------- |
| 0         | 初始化值                                                     |
| CANCELLED | 1，表示线程获取锁的请求已经取消了                            |
| SIGNAL    | -1，等待被唤醒了，准备请求锁                                 |
| CONDITION | -2，在队列中等待                                             |
| PROPAGATE | -3，当前线程节点的后续节点的acquireShare方法能够被无条件执行 |


#### AQS的公平锁与非公平锁实现

主要是判断当前线程是否位于同步队列中的第一个。如果是则返回true，否则返回false。

非公平锁：

```java
final void lock() {  // 直接CAS操作上锁  if (compareAndSetState(0, 1))    setExclusiveOwnerThread(Thread.currentThread());  else    acquire(1);}
```

