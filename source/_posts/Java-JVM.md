---
title: Java-JVM
date: 2022-01-08 15:45:22
tags: Java
cover: /images/JavaLogo.png
---

# Java虚拟机学习

[TOC]



## Java内存模型

### 程序计数器

当前线程的所执行字节码行号指示器。

### Java虚拟机栈

Java方法运行的地方

每个方法运行时都有一个栈帧，**栈帧**保存有

- 局部变量表
- 操作数栈
- 动态链接
- 方法出口

### 本地方法栈

native方法运行的地方

和Java虚拟机栈很相似

### Java堆 / GC年轻代和老年代

**线程共享**

Java对象生存的地方，也是GC主要发生的地方。

### Java方法区 / GC永久代

**线程共享**

用来储存已被虚拟机加载的类信息，常量，静态变量等数据。

#### 运行时常量池

属于Java方法区的一部分（Java8以前）

存放编译时期生成的各种字面量和符号引用。

#### String.intern()

1.6以前intern()会把首次出现的字符串复制到永久代，返回永久代对象。
1.6以后intern()不会把首次出现的字符串复制到永久代，直接返回对象引用。

#### Integer和StringBuilder()和“字符串”

Integer -128~127 永久代中保存。

"字符串" 如果永久代中存在，直接取，否则创建对象。

### 元空间

Java8后 Java方法区被元空间代替。

元空间使用直接内存。不受虚拟机限制。

为什么要用元空间代替方法区？

- 整个永久代有一个 JVM 本身设置固定大小上限，无法进行调整，而元空间使用的是直接内存，受本机可用内存的限制，虽然元空间仍旧可能溢出，但是比原来出现的几率会更小。
- 元空间大小可以动态调整。
- 在 JDK8，合并 HotSpot 和 JRockit 的代码时, JRockit 从来没有一个叫永久代的东西, 合并之后就没有必要额外的设置这么一个永久代的地方了。

### 逃逸分析技术

测试 逃逸

```java
// jvm参数：-Xmx10m -Xms10m -XX:+PrintGC -XX:-DoEscapeAnalysis
public class Main {
    public static void main(String[] args) {
        while (true) {
            Integer integer = new Integer(1111111);
        }
    }
}
```

#### 逃逸分析算法

使用连通图来构建对象和对象引用之间的可达性关系，并在次基础上，提出一种组合数据流分析法。

#### 逃逸状态

- 全局逃逸：一个对象的作用范围跳出了当前方法或线程。

  例子：对象作为返回值，静态变量。

- 参数逃逸：一个对象被方法的参数传递

- 没有逃逸：当前方法中对象没有发送逃逸

#### 逃逸分析优化

- 锁消除：如果检测到对象未逃逸，那么锁是无效的，编译器优化就会移出掉这些操作。
- 标量替换：如果一个对象没有发生逃逸，那么根本不需要在堆中创建它，只需要在栈中创建它所用到的成员标量即可。
- 栈上分配：同上，对象的标量分配在栈中，随方法的结束销毁，减少了GC压力。

#### 虚拟机参数

在JDK1.6后才有逃逸分析的实现

开启：`-XX:+DoEscapeAnalysis`

关闭：`-XX:-DoEscapeAnalysis`

## 对象

### Java对象内存布局

HotSpot虚拟机中，对象在内存的储存布局分三块区域：对象头，内容，对齐填充。

#### 对象头

对象头有两部分，Mark Word和指针类型。

1. 对象类型指针 **8/4字节**（地址）

   当JVM开启压缩指针：4字节（默认），否则为8字节

   对象指向他的类元数据的指针。虚拟机通过这个指针确定这个对象属于哪个类的实例。

2. 标记Mark Word **8字节**

   储存对象自身运行时数据，如hashCode、GC年龄、锁状态、线程持有的锁等。

   储存内容(29bit) - 标志位(2) - 0(1bit)

3. 数组[] **4字节**（如果是个数组）

   记录数组长度

#### 内容

对象真正的有效信息，就是代码中定义的各个字段和字段的内容。

#### 对齐填充

HotSpot虚拟机规定对象的大小必须是8字节（32位）的整数倍。

#### 如何计算对象的大小？

工具依赖：

```xml
<dependency>
  <groupId>org.apache.lucene</groupId>
  <artifactId>lucene-core</artifactId>
  <version>4.0.0</version>
</dependency>
```

```java
Integer i = 1;
System.out.println(RamUsageEstimator.shallowSizeOf(i)); // 16（4 + 8 + 4）
```

手动计算：

1. 计算头部：类型4/8 + MarkWord8，如果是数组再+4。

2. 计算内容部分

   从父类开始寻找非静态变量。找到一个对象类型则加4/8，找到主数据类型则+4/8。

#### new对象在堆上分配的时候，会发生线程安全问题吗？

会造成线程安全问题！概念上来说，需要给堆加锁。但是由于这样做效率太低，为了减小锁粒度，HotSpot用TLAB技术优雅地解决了此问题。

HotSpot的实现：

**TLAB**：线程本地分配缓存

Java虚拟机在新生代Eden空间分配了一小块**线程私有**的内存空间TLAB。

新创建的对象优先在这个空间分配，他们不存在线程共享也适合快速GC。

### 对象的访问定位

1. 句柄访问

   虚拟机栈中存在对象的引用reference，引用储存对象的句柄地址。

   句柄池包含**到对象实例数据的指针**和**到对象类型数据的指针**

2. 直接指针（HotSpot虚拟机实现）

   虚拟机栈中（方法栈）存在对象的引用reference，引用储存的对象的直接访问地址。

   对象类型指针，可以从对象头部中获取。

## GC垃圾收集

#### 说明

**程序计数器，虚拟机栈，本地方法栈**3个区域随线程生死。

当方法或线程结束，内存自然也回收了。

而**堆**和**方法区**的内存都是动态回收的。

#### 如何判断对象生死？

1. 引用计数法

   给对象添加一个引用计数器，有地方引用它时，计数器+1；引用失败时，计数器-1；

   当计数器为0，判断对象死。

   存在问题：难以解决对象之间循环引用问题。

2. 可达性分析法

   通过一系列**GC Root**对象作为起点，向下搜索，搜索路径称**引用链**。

   当一个对象从**GC Root**不可达，证明这个对象是不可引用的。

#### 可作为GC Root的对象

- 虚拟机栈中引用的对象
- 本地方法栈中引用的对象
- 方法区中的静态变量、常量引用的对象

#### 引用类型

jdk1.2后，引用类型分为强引用、软引用、弱引用、虚引用四种。强度逐级减弱。

- 强引用

  代码中普遍存在的，`Object o = new Object()`，只要强引用还在，就不会回收。

- 软引用

  描述还有用但非必须对象。内存溢出之前，会进行第二次回收。

  `SoftReference s = new SoftReference() `

  用途：缓存

- 弱引用

  非必须对象，比软引用更弱。只能存活到下一次垃圾收集发生之前。

  `WeakReference w = new WeakReference()`

  用途：threadLocal

- 虚引用/幻影引用

  最弱的引用，存在目的：能在这个对象被垃圾收集器回收时收到一个系统通知。

  `PhantomReference<Object> pf = new PhantomReference<Object>(obj, null);`

  用途：通知

### 堆的收集

#### 垃圾收集算法

- 标记-清除

  用GC Root进行可达性分析，标记，清除对象。

  缺点：标记和清除的效率都不高，容易产生不连续内存碎片。

- 标记-整理

  让存活的对象都向一端移动，然后清理掉端边界以外的内存。

  缺点：大量移动对象，效率不高。

- 复制

  把内存分为两块，一块内存用完了，就把存活的复制到另一块。

  HotSpot实现：一块较大的Eden和两块较小的Survivor。8：1的比例。

  每次使用Eden和一块Survivor，回收时，把存活的复制到另一块Survivor上。清理原来的Eden和Survivor。

#### 分代收集

- 新生代Young：**复制**

- 老年代Tenured：标记-清除，标记-整理

#### 何时收集？什么时候Full GC 什么时候Minor GC

在SafePoint，程序可以进入GC。

老年代不足时，Full GC，伴随一次Minor GC

新生代不足时，Minor GC

#### 垃圾收集器

##### 新生代收集器

新生代收集器有Serial，ParNew和PS三种。

- Serial

  单线程收集器。

- ParNew

  Serial的多线程版本。

- PS Parallel Scavenge （客户端常用）

  注重吞吐量的垃圾收集器，意味着GC的平均时间短

  吞吐量 = 代码运行时间/(代码运行时间 + 垃圾收集时间)

##### 老年代收集器

老年代收集器有Serial Old，Parallel Old和CMS三种。

- Serial Old

  单线程收集器老年代版本。

- Parallel Old

  PS收集器老年代版本，多线程并发收集，使用标记-整理算法。

- CMS Concurrent Mark Sweep

  一种以获取最短暂停时间为目标的收集器。

  收集步骤：

  - 初始标记 stop

    标记GC Root

  - 并发标记

    从GC Root 开始扫描

  - 重新标记 stop

    修正因扫描而产生变动对象的记录

  - 并发清除 

    标记-清除

  其中**并发标记**和**并发清除**可以和用户程序并发执行，**初始标记**和**重新标记**需要stop the world。

  缺点：

  - 对CPU资源敏感，因为需要多线程。
  - 无法处理浮动垃圾，指的是标记完成后产生的垃圾。
  - 基于 标记-清除 算法实现，可能产生内存碎片。

##### G1 收集器

最前沿的收集器，范围是整个新生代和老年代。

新生代和老年代不再是物理隔离，他们都是一部分Region(不需要连续)的集合。

把对内存分为多个Region空间。每个Region可以单独回收。

特点：

- 并行与并发
- 分代收集
- 空间整合
- 可预测的停顿

##### 根据Region划分的内存区域，如何判断对象存活？

各个Region中的对象互相引用，如何判断关系？

为了避免全堆扫描，G1中的每个Region都有一个Remembered Set。

当发生对Reference类型数据进行写操作时，产生一个Write Barrier中断，检查引用的对象是否处于不同Region，如果是则通过CardTable把相关引用信息记录到所属Region的Remembered Set之中。

这样一来，就不必进行全堆扫描，也不会造成遗漏。

##### 运作流程

和CMS的流程很相似。

- 初始标记 GC Root 扫描标记 stop
- 并发标记 GC Root 可达性分析
- 最终标记 微调，修正因并发标记期间程序继续运作改变的标记 stop
- 筛选回收 回收，根据各个Region的回收价值和成本进行排序回收 stop

### 方法区/永久代Perm收集

主要回收废弃常量和无用的类。

主要发生在类卸载。

1. 废弃常量

   从一系列GC Root不可达常量池的某个常量引用，那么这个常量就会被清理。

2. 无用的类

   三个条件：

   - 该类所有实例已经被回收。
   - 加载该类的ClassLoader已经被回收。
   - 该类的Class对象没有在任何地方被引用。

### 内存分配与回收策略

- Minor GC：新生代回收
- Major GC/Full GC: 新生代和老年代一起回收。

#### 分配

1. 对象优先在Eden分配

2. 大对象直接进入老年代

3. 长期存活对象将进入老年代

   对象每熬过一次Minor GC年龄增加1岁。默认15岁会到老年代。

#### 年龄判定

如果Survivor空间中相同年龄的对象大小大于Survivor的一半，那么年龄大于等于该年龄的对象就进入老年代。

## 实战

### 堆溢出OutOfMemoryError异常

创建尽可能多的对象并维持对象的GC root，造成堆内存溢出。

VM参数：-Xms20m -Xmx20m -XX:+HeapDumpOnOutOfMemoryError

### 栈溢出StackOverflowError 异常

1. 请求栈的深度大于虚拟机允许的深度——栈溢出
2. 创建多个线程，线程所用的空间太大，导致进程空间溢出——内存溢出

VM参数：-Xss256k

## 虚拟机参数

-Xms:设置堆最小值

-Xmx:设置堆最大值

-Xss:设置虚拟机栈大小。

-XX:MaxMetaspaceSize 设置元空间大小。

-XX:+HeapDumpOnOutOfMemoryError: OutOfMemoryError异常时，Dump出堆储存快照。

-XX:+PrintGCDetails 打印GC日志。

## Java类与类加载机制

每个java程序都是一个JVM进程（不同的main运行在不同JVM中）。

#### 如何确定一个类和另一个类相同？

- 该类的全限定名 

  查看该类的全限定名：xxx.getClass().getName()

- 该类的类加载器 

  查看该类是哪个类加载器加载的：clazz.getClassLoader()

### 类加载时机

- 遇到new、getstatic、putstatic或invokestatic这四条字节码指令。
- 反射，通过java.lang.reflect包对类进行反射调用时。
- 初始化一个类，发现父类未被初始化时。
- 虚拟机启动时，包含main的那个启动类。

### 双亲委派机制（并不是强制约束，是一种思想）

#### 核心思想

类加载器自己拿到类名后，先交给父类加载，当父类不能加载时，自己才尝试去加载这个类。

#### 优点

Java中的类随着它的类加载器一起具备了一种带优先级的层次关系。

如：就算自己编写的类名和系统类的类名一模一样，那么JVM也能正确加载系统类。

- 保证了系统类的唯一
- 保证了已经加载的类不会重复加载

#### 缺点

如果系统类要回调用户类怎么办？（高层调用低层）

#### 什么时候需要打破双亲委派机制

- 包的版本冲突。
- 层级高的类需要使用层级低的类。

### 三大类加载器

这三大类加载器都是ClassLoader实例。

1. Bootstrap ClassLoader：启动类加载器，负责加载JAVA_HOME/lib中的东东。

   由于此加载器是C++/C实现的，所以Java中可能返回null。

2. Extension ClassLoader：扩展类加载器，负责加载JAVA_HOME/lib/ext目录中的东东。

3. Application ClassLoader：应用程序类加载器/系统类加载器，负责加载用户类路径ClassPath下的东东（就是自己写的程序）。

#### 源码分析

通过父加载器parent可以看出，双亲委派机制的实现采用组合，而不是继承。

关键类：

- ClassLoader ：一个负责加载类的类。需要自定义类加载器的时候，继承它，里面有许多内置方法可用。

关键方法：

- findLoadedClass 查看该类有没有被加载
- findClass 通过某种方式（用户自定义的关键），找到class文件，并转换成Class
- defineClass 把class文件转换成Class（实际的加载方法，是native的）

分析完后发现，自定义一个UserClassLoader需要：

- 继承ClassLoader

```java
protected Class<?> loadClass(String name, boolean resolve)
        throws ClassNotFoundException
    {
        synchronized (getClassLoadingLock(name)) {
            // 首先，查看该类有没有被加载。
            Class<?> c = findLoadedClass(name);
            if (c == null) {
                long t0 = System.nanoTime();
              // 尝试委派给父类加载器parent
                try {
                    if (parent != null) {
                        c = parent.loadClass(name, false);
                    } else {
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    // 父类加载器不能加载此类
                }

                if (c == null) {
                    // 如果父类加载不了，自己加载
                    long t1 = System.nanoTime();
                    c = findClass(name); // 用户类加载器重写关键方法，如何find

                    // this is the defining class loader; record the stats
                    sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                    sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                    sun.misc.PerfCounter.getFindClasses().increment();
                }
            }
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
    }
```

#### 当自定义一个类加载器的时候，我们在自定义什么？

简单地说，其实是为了自定义findClass方法，为了能自定义寻找类的方法。

比如Tomcat，有一大堆类加载器。

Tomcat本是别人写好的程序，不知道用户编写的逻辑实现类放在哪。

所以，Tomcat自定义一些类加载器，规定一些路径。只要用户按照规定放置类，Tomcat就能找到并加载用户的类。

#### 自定义类加载器的正确姿势！

委派模型（更上层省略）：LoaderChild -> LoaderFather -> Application ClassLoader

核心：组合而不是继承。

自定义一个类加载器方法：

```java
@Override
protected Class<?> findClass(String name) throws ClassNotFoundException {
  System.out.println("Parent:" + this.getParent());
  System.out.println("name = " + name);
  // 双亲委派
  byte[] classData = getClassData(name);
  if (classData == null) {
    throw new ClassNotFoundException();
  }
  else {
    // 父类加载器不能加载，自己加载
    return defineClass(name, classData, 0, classData.length);
  }
}
```



自定义父子类加载器方法：

1. LoaderFather 继承 ClassLoader，重写findClass方法，在findClass方法里加入defineClass()。

   关键：继承ClassLoader，这样一来，LoaderFather的parent就是 Application ClassLoader了（前面有分析）。

2. LoaderChild 继承 ClassLoader，但是！**构造函数**要填**LoaderFather**，这样就成功地组合了父类加载器。

   剩下部分和前者一样。

#### 自定义ClassLoader Demo

> 直接看Demo吧，这上下文意识流样的废话太多了

https://github.com/sawyerRick/MyClassContainer

### 总结

#### 如果想打破双亲委派机制：

重写loadClass方法

#### 如果想不想打破双亲委派机制：

重写findClass方法

#### 如何优雅地defineClass？

使用javaassist

#### 问题：ClassLoader此抽象类的parent从哪来？

自给自足，如果不给它添加parent，它就自己添加**getSystemClassLoader()**，也就是应用程序类加载器/系统类加载器

```java
protected ClassLoader() {
    this(checkCreateClassLoader(), getSystemClassLoader());
}
```

## 杂谈

#### 如何判断程序内存泄漏？

- 使用jvisualvm分析工具，查看Visual GC标签（插件，默认没有，需要安装）。

  分析堆内存情况，如果老年代内存不足，GC发生频繁（GC频率可以通过jconsole工具分析）那么很可能发生了内存泄漏。

#### 如何定位内存泄漏？

- 使用jvisualvm分析工具，转储堆内存快照（dump堆），分析堆中实例数量，如果某个实例数量很多，很可能就是此实例泄漏。

#### 如何合理设置线程数量？

- 线程数量不能太多，也不能太少，不需要计算出精确的数量。

- 根据公式：cpu数量 * 期望cpu使用率 * (1 + 等待时间/ 计算时间)

  公式分析：如果任务是IO密集型，应该开较多的线程；如果任务是计算密集型，开再多线程也没用，反而会因上下文的切换耗费更多的资源，所以应该开较少的线程。

  一般来说，开和cpu逻辑核心数相同的线程数量即够用。

## 工具

### Jdk自带

jvisualvm：可以查看内存快照

jconsole: 查看

jps: 查看正在运行的虚拟机进程。

- 反解析字节码：javap -verbose xxxxx

  把字节码解析为code区

### JVM参数

- 堆大小 最大：`-Xmx10m` ；初始： `-Xms10m`
- 栈内存大小：`-Xss10m`
- 堆新生代老年代比例：`-XX:NewRatio`
- 方法区（JDK1.8以前永久代）：初始：`-XX: PermSize=128m` 最大：`-XX:MaxPermSize=512m` 
- 元数据空间：初始：`-XX:MetaspaceSize=128m` 最大： `-XX:MaxMetaspaceSize=512m`
- 输出GC日志：-XX:+PrintGC
- 逃逸分析开启/关闭： -XX:-DoEscapeAnalysis

 
