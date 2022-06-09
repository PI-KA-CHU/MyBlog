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

![image-20220609130657476](https://raw.githubusercontent.com/PI-KA-CHU/Image-OSS/main/images/image-20220609130657476.png)

#### 2.4.1 分代收集器

（分代收集指在物理层面分成新生代和老年代）

- **Serial**：【早期的单线程新生代收集器，采用复制算法】
- **Serial Old**：【Serial的老年代收集器，采用标记-整理算法】
- **ParNew：**【新生代，采用“复制“算法】一款多线程的收集器，是Serial的优化版，采用复制算法，主要工作在 Young 区，可以通过 `-XX:ParallelGCThreads` 参数来控制收集的线程数，整个过程都是 STW 的，常与 CMS 组合使用。
- **Parallel Scavenge**：【新生代，多线程 - ”复制“算法，以获取更大的吞吐量为目的】
  - 吞吐量 = 运行用户代码时间 / （运行用户代码时间 + 垃圾回收时间）
- **Parallel Old**：【老年代，多线程 - ”标记-整理“算法，以获取更大的吞吐量为目的】
- **CMS：**【老年代：采用“**标记-清除**”算法，以获取最短回收停顿时间为目标】，分 4 大步进行垃圾收集，其中初始标记和重新标记会 STW ，多数应用于互联网站或者 B/S 系统的服务器端上，JDK9 被标记弃用，JDK14 被删除，详情可见 [JEP 363](https://openjdk.java.net/jeps/363)。
  - 初始标记：标记GC Root的直接关联对象（STW）
  - 并发标记：开始对象图的并发标记
  - 重新标记：标记由于并发标记过程中用户线程活动产生的对象（STW）
  - 并发清除：并发清除垃圾对象（由于清除过程用户线程仍在活动，所以有“浮动垃圾”产生）


#### 2.4.2 分区收集器

（新生代和老年代的概念仍然存在，但是已经不在物理上划分）

- **G1**：【新生老年代：新生采用“**复制**”算法，老年采用“**标记-整理**”算法，以获取最短回收停顿时间为目标】一种服务器端的垃圾收集器，应用在多处理器和大容量内存环境中。作为JDK9的默认GC，不强制在物理内存上划分分代，而是面向堆内存进行管理，堆内存被划分未多个region（1-32M），而不同的region可能扮演不同的角色，G1根据其角色采用不同的回收策略。大致步骤是：计算region的回收价值（垃圾多少及回收时间）并维护优先列表，根据用户设置的允许停顿时间，优先处理回收价值高的region。
  - 初始标记：

- **ZGC：**JDK11 中推出的一款低延迟垃圾回收器，适用于大内存低延迟服务的内存管理和回收，SPECjbb 2015 基准测试，在 128G 的大堆下，最大停顿时间才 1.68 ms，停顿时间远胜于 G1 和 CMS。
- **Shenandoah：**由 Red Hat 的一个团队负责开发，与 G1 类似，基于 Region 设计的垃圾收集器，但不需要 Remember Set 或者 Card Table 来记录跨 Region 引用，停顿时间和堆的大小没有任何关系。停顿时间与 ZGC 接近，下图为与 CMS 和 G1 等收集器的 benchmark。



&nbsp;

## 三、OOM实战



如何获取堆和栈的dump文件？（dump为JVM运行时的快照）

- jmap命令获取堆dump
- jstack命令获取线程调用栈dump





## 参考

- ZGC：https://mp.weixin.qq.com/s/RFwXYdzeRkTG5uaebVoLQw
- GC：https://mp.weixin.qq.com/s/RFwXYdzeRkTG5uaebVoLQw