author = "pikachu"
title = "JVM"
date = "2018-10-10"
description = " JMM、GC、OOM"
tags = [
    "jvm"
]
categories = [
    "it","java"
]

+++

&nbsp;

目标：

- 哪些区域会发生OOM异常？
- 垃圾回收器有哪些？

&nbsp;

## 一、JMM模型

![图片](https://raw.githubusercontent.com/PI-KA-CHU/Image-OSS/main/images/640)

- GC 主要工作在 Heap 区和 MetaSpace 区（上图蓝色部分）

&nbsp;

## 二、GC垃圾回收

- 9种常见GC：https://mp.weixin.qq.com/s/RFwXYdzeRkTG5uaebVoLQw

![image-20220607112223300](https://raw.githubusercontent.com/PI-KA-CHU/Image-OSS/main/images/image-20220607112223300.png)

### 2.1 GC基础

- **GC：**GC 本身有三种语义，下文需要根据具体场景带入不同的语义：

- - **Garbage Collection**：垃圾收集技术，名词。
  - **Garbage Collector**：垃圾收集器，名词。
  - **Garbage Collecting**：垃圾收集动作，动词。

- **Mutator：**生产垃圾的角色，也就是我们的应用程序，垃圾制造者，通过 Allocator 进行 allocate 和 free。

- **TLAB**：Thread Local Allocation Buffer 的简写，基于 CAS 的独享线程（Mutator Threads）可以优先将对象分配在 Eden 中的一块内存，因为是 Java 线程独享的内存区没有锁竞争，所以分配速度更快，每个 TLAB 都是一个线程独享的。

- **Card Table：**中文翻译为卡表，主要是用来标记卡页的状态，每个卡表项对应一个卡页。当卡页中一个对象引用有写操作时，写屏障将会标记对象所在的卡表状态改为 dirty，卡表的本质是用来解决跨代引用的问题。具体怎么解决的可以参考 StackOverflow 上的这个问题 [how-actually-card-table-and-writer-barrier-works](https://stackoverflow.com/questions/19154607/how-actually-card-table-and-writer-barrier-works)，或者研读一下 cardTableRS.app 中的源码。

&nbsp;

### 2.2 分配对象

Java 中对象地址操作主要使用 Unsafe 调用了 C 的 allocate 和 free 两个方法，分配方法有两种：

- **空闲链表（free list）：**通过额外的存储记录空闲的地址，将随机 IO 变为顺序 IO，但带来了额外的空间消耗。
- **碰撞指针（bump  pointer）：**通过一个指针作为分界点，需要分配内存时，仅需把指针往空闲的一端移动与对象大小相等的距离，分配效率较高，但使用场景有限。



### 2.3 收集对象

#### 2.3.1 识别垃圾

- **引用计数法（Reference Counting）**：对每个对象的引用进行计数，每当有一个地方引用它时计数器+1，引用失效则-1，引用计数器放在对象头中。虽然循环引用问题可以通过Recycler算法解决，但是在多线程环境下，引用计数变更需要进行同步操作，性能较低，早期的编程语言才采用此算法。
- **可达性分析，又称引用链法（Tracing GC）：**从 GC Root 开始进行对象搜索，可以被搜索到的对象即为可达对象，整个连通图之外的对象便可以作为垃圾被回收掉。目前 Java 中主流的虚拟机均采用此算法。

&nbsp;

#### 2.3.2 内存回收算法

自从有自动内存管理出现之时就有的一些收集算法，不同的收集器也是在不同场景下进行组合。

- **Mark-Sweep（标记-清除）：**【基础算法】回收过程主要分为两个阶段，第一阶段为追踪（Tracing）阶段，即从 GC Root 开始遍历对象图，并标记（Mark）所遇到的每个对象，第二阶段为清除（Sweep）阶段，即回收器检查堆中每一个对象，并将所有未被标记的对象进行回收，整个过程不会发生对象移动。整个算法在不同的实现中会使用三色抽象（Tricolour Abstraction）、位图标记（BitMap）等技术来提高算法的效率，存活对象较多时较高效。
- **Copying（复制）：**【优化：*新生代*，新生代存活时间短，不需要过多复制】将空间分为两个大小相同的 From 和 To 两个半区，同一时间只会使用其中一个，每次进行回收时将一个半区的存活对象通过复制的方式转移到另一个半区。有递归（Robert R. Fenichel 和 Jerome C. Yochelson提出）和迭代（Cheney 提出）算法，以及解决了前两者递归栈、缓存行等问题的近似优先搜索算法。复制算法可以通过碰撞指针的方式进行快速地分配内存，但是也存在着空间利用率不高的缺点，另外就是存活对象比较大时复制的成本比较高。
- **Mark-Compact （标记-整理）：**【优化：*老年代*，老年代存活时间长，复制算法需要较多的复制操作，效率较低】这个算法的主要目的就是解决在非移动式回收器中都会存在的碎片化问题，也分为两个阶段，第一阶段与 Mark-Sweep 类似，第二阶段则会对存活对象按照整理顺序（Compaction Order）进行整理。主要实现有双指针（Two-Finger）回收算法、滑动回收（Lisp2）算法和引线整理（Threaded Compaction）算法等。

![image-20220607124816646](https://raw.githubusercontent.com/PI-KA-CHU/Image-OSS/main/images/image-20220607124816646.png)

&nbsp;

### 2.4 垃圾收集器

- 垃圾回收器是回收算法的具体实现，不同的回收器关注不用多特性，即作用在不同的场景。

#### 2.4.1 分代收集器

（分代收集指在物理层面分成新生代和老年代）

- **Serial**：【早期的单线程新生代收集器，采用复制算法】
- **Serial Old**：【Serial的老年代收集器，采用标记-整理算法】
- **ParNew：**【新生代，采用“复制“算法】一款多线程的收集器，是Serial的优化版，采用复制算法，主要工作在 Young 区，可以通过 `-XX:ParallelGCThreads` 参数来控制收集的线程数，整个过程都是 STW 的，常与 CMS 组合使用。
- **Parallel Scavenge**：【新生代，多线程 - ”复制“算法，以获取更大的吞吐量为目的】
  - 吞吐量 = 运行用户代码时间 / （运行用户代码时间 + 垃圾回收时间）

- **Parallel Old**：【老年代，多线程 - ”标记-整理“算法，以获取更大的吞吐量为目的】
- **CMS：**【老年代：采用“**标记-清除**”算法，以获取最短回收停顿时间为目标】，分 4 大步进行垃圾收集，其中初始标记和重新标记会 STW ，多数应用于互联网站或者 B/S 系统的服务器端上，JDK9 被标记弃用，JDK14 被删除，详情可见 [JEP 363](https://openjdk.java.net/jeps/363)。

#### 2.4.2 分区收集器

（新生代和老年代的概念仍然存在，但是已经不在物理上划分）

- **G1**：【新生老年代：新生采用“**复制**”算法，老年采用“**标记-整理**”算法，以获取最短回收停顿时间为目标】一种服务器端的垃圾收集器，应用在多处理器和大容量内存环境中。
- **ZGC：**JDK11 中推出的一款低延迟垃圾回收器，适用于大内存低延迟服务的内存管理和回收，SPECjbb 2015 基准测试，在 128G 的大堆下，最大停顿时间才 1.68 ms，停顿时间远胜于 G1 和 CMS。
- **Shenandoah：**由 Red Hat 的一个团队负责开发，与 G1 类似，基于 Region 设计的垃圾收集器，但不需要 Remember Set 或者 Card Table 来记录跨 Region 引用，停顿时间和堆的大小没有任何关系。停顿时间与 ZGC 接近，下图为与 CMS 和 G1 等收集器的 benchmark。



&nbsp;

## 三、GC+OOM实战



### 3.1 JVM常用工具



#### 标准命令行终端

（i have 2 polices man）

如何获取堆和栈的dump文件？（dump为JVM运行时的快照）

- jmap：获取堆dump
- jstack：获取线程调用栈dump
- jps：
- jinfo：
- jstat：
- jstack：
- jmap：



#### 整合型命令行终端

- jcmd
- arthas
- vjtools



#### 可视化界面

- JConsole
- JVisualvm
- JProfiler（进阶）



### 3.2 GC实战



#### 判断GC问题核心指标

- **延迟（Latency）**：也可以理解为最大停顿时间，即垃圾收集过程中一次 STW 的最长时间，越短越好，一定程度上可以接受频次的增大，GC 技术的主要发展方向。
- **吞吐量（Throughput）：**应用系统的生命周期内，由于 GC 线程会占用 Mutator 当前可用的 CPU 时钟周期，吞吐量即为 Mutator 有效花费的时间占系统总运行时间的百分比，例如系统运行了 100 min，GC 耗时 1 min，则系统吞吐量为 99%，吞吐量优先的收集器可以接受较长的停顿。



#### GC日志分析

1. 打印GC日志
   - `-XX:+PrintGCDetails`：表示的是打印GC日志详情
   - `-XX:+PrintGCTimeStamps`：表示打印GC时间戳
   - `-Xloggc: ./gc.log`：表示在当前目录下生成gc.log文件

```
-XX:+PrintGCDetails -XX:+PrintGCTimeStamps -Xloggc:./gc.log
```



2. gceasy在线工具：https://blog.csdn.net/CoderBruis/article/details/101234738

![图片](https://raw.githubusercontent.com/PI-KA-CHU/Image-OSS/main/images640.png)



#### 重点GC Case

- **System.gc()：**手动触发GC操作
- **CMS：**CMS GC 在执行过程中的一些动作，重点关注 CMS Initial Mark 和 CMS Final Remark 两个 STW 阶段
- **Promotion Failure：**Old 区没有足够的空间分配给 Young 区晋升的对象（即使总可用内存足够大）
- **Concurrent Mode Failure：**CMS GC 运行期间，Old 区预留的空间不足以分配给新的对象，此时收集器会发生退化，严重影响 GC 性能，下面的一个案例即为这种场景
- **GCLocker Initiated GC：**如果线程执行在 JNI 临界区时，刚好需要进行 GC，此时 GC Locker 将会阻止 GC 的发生，同时阻止其他线程进入 JNI 临界区，直到最后一个线程退出临界区时触发一次 GC



## 参考

- ZGC：https://mp.weixin.qq.com/s/RFwXYdzeRkTG5uaebVoLQw
- GC：https://mp.weixin.qq.com/s/RFwXYdzeRkTG5uaebVoLQw