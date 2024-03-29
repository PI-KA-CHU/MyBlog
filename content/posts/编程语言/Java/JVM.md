

+++

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

![image-20220622231028306](https://raw.githubusercontent.com/PI-KA-CHU/Image-OSS/main/imagesimage-20220622231028306.png)

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
- **可达性分析，又称引用链法（Tracing GC）：**从 GC Root （两栈两方法：JVM栈、本地方法栈、方法区的常量池、方法区的静态属性）开始进行对象搜索，可以被搜索到的对象即为可达对象，整个连通图之外的对象便可以作为垃圾被回收掉。目前 Java 中主流的虚拟机均采用此算法。

&nbsp;

#### 2.3.2 内存回收算法

自从有自动内存管理出现之时就有的一些收集算法，不同的收集器也是在不同场景下进行组合。

- **Mark-Sweep（标记-清除）：**【基础算法】回收过程主要分为两个阶段，第一阶段为追踪（Tracing）阶段，即从 GC Root 开始遍历对象图，并标记（Mark）所遇到的每个对象，第二阶段为清除（Sweep）阶段，即回收器检查堆中每一个对象，并将所有未被标记的对象进行回收，整个过程不会发生对象移动。整个算法在不同的实现中会使用三色抽象（Tricolour Abstraction）、位图标记（BitMap）等技术来提高算法的效率，存活对象较多时较高效。
- **Copying（复制）：**【优化：*新生代*，新生代存活时间短，不需要过多复制】将空间分为两个大小相同的 From 和 To 两个半区，同一时间只会使用其中一个，每次进行回收时将一个半区的存活对象通过复制的方式转移到另一个半区。有递归（Robert R. Fenichel 和 Jerome C. Yochelson提出）和迭代（Cheney 提出）算法，以及解决了前两者递归栈、缓存行等问题的近似优先搜索算法。复制算法可以通过碰撞指针的方式进行快速地分配内存，但是也存在着空间利用率不高的缺点，另外就是存活对象比较大时复制的成本比较高。
- **Mark-Compact （标记-整理）：**【优化：*老年代*，老年代存活时间长，复制算法需要较多的复制操作，效率较低】这个算法的主要目的就是解决在非移动式回收器中都会存在的碎片化问题，也分为两个阶段，第一阶段与 Mark-Sweep 类似，第二阶段则会对存活对象按照整理顺序（Compaction Order）进行整理。主要实现有双指针（Two-Finger）回收算法、滑动回收（Lisp2）算法和引线整理（Threaded Compaction）算法等。

![image-20220607124816646](https://raw.githubusercontent.com/PI-KA-CHU/Image-OSS/main/images/image-20220607124816646.png)

&nbsp;

### 2.4 垃圾收集器

- 垃圾回收器是回收算法的具体实现，不同的回收器关注不用多特性，即作用在不同的场景
- 垃圾回收器的衡量指标：**内存占用**、**吞吐量**和**延迟**

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

- **G1**：【新生老年代：新生采用“**复制**”算法，老年采用“**标记-整理**”算法，以控制回收停顿时间并尽可能的提高吞吐量为目标】一种服务器端的垃圾收集器，应用在多处理器和大容量内存环境中。作为JDK9的默认GC，不强制在物理内存上划分分代，而是面向堆内存进行管理，堆内存被划分未多个region（1-32M），而不同的region可能扮演不同的角色，G1根据其角色采用不同的回收策略。大致步骤是：计算region的回收价值（垃圾多少及回收时间）并维护优先列表，根据用户设置的允许停顿时间，优先处理回收价值高的region。
  - 初始标记：标记GC Root直接关联到的对象（STW）
  - 并发标记：对第一步扫描到的对象进行可达性分析，递归扫描整个堆里的对象图。
  - 最终标记：标记由于并发标记过程中用户线程活动产生的对象（STW）
  - 筛选回收：负责更新Region的统计数据，对各个Region的回 收价值和成本进行排序，根据用户所期望的停顿时间来制定回收计划。回收过程如下：将要回收的region复制到空的region-清空原来的region。（STW）
  - 相比于CMS
    - G1有着无空间碎片、可均衡停顿时间和吞吐量的优势，但是其垃圾收集产生的内存占用及其他额外负载都比CMS高
    - CMS在小内存的表现优于G1，G1在大内存应用上能发挥更大的优势（堆容量平衡点：6G-8G）

- **Shenandoah：**由 Red Hat 的一个团队负责开发，与 G1 类似，基于 Region 设计的垃圾收集器，但不需要 Remember Set 或者 Card Table 来记录跨 Region 引用，停顿时间和堆的大小没有任何关系。停顿时间与 ZGC 接近，下图为与 CMS 和 G1 等收集器的 benchmark。
- **ZGC：**【新生老年代，“**标记-整理**”算法，采用**染色指针技术**，极大的降低停顿时间】JDK11 中推出的一款低延迟垃圾回收器，适用于大内存低延迟服务的内存管理和回收，SPECjbb 2015 基准测试，在 128G 的大堆下，最大停顿时间才 1.68 ms，停顿时间远胜于 G1 和 CMS。

&nbsp;

## 三、GC+OOM实战



### 3.1 JVM常用工具



#### 标准命令行终端

（i have 2 polices man）

如何获取堆和栈的dump文件？（dump为JVM运行时的快照）

- jmap：获取堆内存快照（heap dump）
- jstack：获取虚拟机的线程快照（stack dump），定位线程出现长时间停顿的原因，如*线程间死锁、死循环、请求外部资源*导致的长时间等待
- jinfo：显示和调整虚拟机的配置参数
- jstat：用于收集虚拟机各方面的运行数据
- jps：显示制定系统内所有的虚拟机进程



#### 整合型命令行终端

- jcmd
- arthas
- vjtools



#### 可视化界面

- JConsole：Java监控与管理控制台，包括*堆内存使用情况、线程、类、CPU使用情况*等
- VisualVM：多合一故障处理工具
  - 显示进程配置、环境信息（jps、jinfo）
  - 监控程序CPU、GC、堆、方法区以及线程的信息（jstat、jstack）
  - dump以及分析堆快照
  - 方法级的性能分析：找出被调用最多、运行时间最长的方法

- JProfiler（商业版）



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



#### GC问题具体分类

- **Unexpected GC**：意外发生的 GC，实际上不需要发生，我们可以通过一些手段去避免。
  - **Space Shock：**空间震荡问题，参见“场景一：动态扩容引起的空间震荡”。
  - **Explicit GC：**显示执行 GC 问题，参见“场景二：显式 GC 的去与留”。
- **Full GC**：全量收集的 GC，对整个堆进行回收，STW 时间会比较长，一旦发生，影响较大。参见“场景七：内存碎片&收集器退化”。
- **MetaSpace：**元空间回收引发问题，参见“场景三：MetaSpace 区 OOM”。
- **Direct Memory：**直接内存（也可以称作为堆外内存）回收引发问题，参见“场景八：堆外内存 OOM”。
- **JNI：**本地 Native 方法引发问题，参见“场景九：JNI 引发的 GC 问题”。



#### 常见场景分析与解决

![图片](https://raw.githubusercontent.com/PI-KA-CHU/Image-OSS/main/images640-20220622235843314.png)

1. **场景一：动态扩容引起的空间震荡**

- **现象**：服务**刚刚启动时 GC 次数较多**，最大空间剩余很多但是依然发生 GC，这种情况我们可以通过观察 GC 日志或者通过监控工具来观察堆的空间变化情况即可。GC Cause 一般为 Allocation Failure，且在 GC 日志中会观察到经历一次 GC ，堆内各个空间的大小会被调整。

![图片](https://raw.githubusercontent.com/PI-KA-CHU/Image-OSS/main/images640-20220622230646396.png)

- **原因**：在 JVM 的参数中 `-Xms` 和 `-Xmx` 设置的不一致，在初始化时只会初始 `-Xms` 大小的空间存储信息，每当空间不够用时再向操作系统申请，这样的话必然要进行一次 GC。
- **解决**：尽量**将成对出现的空间大小配置参数设置成固定的**，如 `-Xms` 和 `-Xmx`，`-XX:MaxNewSize` 和 `-XX:NewSize`，`-XX:MetaSpaceSize` 和 `-XX:MaxMetaSpaceSize` 等。



2. **场景三：MetaSpace 区 OOM**

- **现象**：JVM 在启动后或者某个时间点开始，**MetaSpace 的已使用大小在持续增长，同时每次 GC 也无法释放，调大 MetaSpace 空间也无法彻底解决**
- **原因**：
  - **MetaSpace 内存管理：**类和其元数据的生命周期与其对应的类加载器相同，只要类的类加载器是存活的，在 Metaspace 中的类元数据也是存活的，不能被回收
  - **MetaSpace 弹性伸缩：**由于 MetaSpace 空间和 Heap 并不在一起，所以这块的空间可以不用设置或者单独设置，一般情况下避免 MetaSpace 耗尽 VM 内存都会设置一个 MaxMetaSpaceSize，在运行过程中，如果实际大小小于这个值，JVM会动态调整其大小。为了避免弹性伸缩带来的额外 GC 消耗，我们会将 `-XX:MetaSpaceSize` 和 `-XX:MaxMetaSpaceSize`两个值设置为固定的，但是这样也会导致在空间不够的时候无法扩容，然后频繁地触发 GC，最终 OOM。所以关键原因就是 ClassLoader 不停地在内存中 load 了新的 Class ，一般这种问题都发生在**动态类加载**等情况上
- **策略**：了解MetaSpace的内存管理后，如何定位和解决就很简单了，可以 dump 快照之后通过 JProfiler 或 MAT 观察 Classes 的 Histogram（直方图） 即可，或者直接通过命令即可定位，看一下具体是哪个包下的 **Class 增加**较多就可以定位了
- **小结**：原理理解比较复杂，但定位和解决问题会比较简单，经常会出问题的几个点有 *Orika 的 classMap、JSON 的 ASMSerializer、Groovy 动态加载类*等，基本都集中在*反射、Javasisit 字节码增强、CGLIB 动态代理、OSGi 自定义类加载器等*的技术点上。另外就是及时给 MetaSpace 区的使用率加一个监控，如果指标有波动提前发现并解决问题。



3. **场景四：过早晋升***

- **现象**：这种场景主要发生在分代的收集器上，90% 的对象朝生夕死，只有在 Young 区经历过几次 GC 的洗礼后才会晋升到 Old 区，每经历一次 GC 对象的 GC Age 就会增长 1，最大通过 `-XX:MaxTenuringThreshold` 来控制。通常可以观察以下几种现象来判断是否发生了过早晋升。

  - 分配速率接近于晋升速率，对象晋升年龄过小。
  - Full GC比较频繁，并且GC后Old区的变化比例很大。

  过早晋升的危害：

  - YGC频繁，总吞吐量下降
  - FGC频繁，有较大停顿时间

  ![图片](https://raw.githubusercontent.com/PI-KA-CHU/Image-OSS/main/images640-20220622231511224.png)

- **原因**：

  - **Young/Eden区过小**：Young区很快被填满并且频繁GC，导致对象年龄快速上升（默认年龄为15），并且被过早晋升到Old区（原本在Young区就可以被回收），而YGC采用的是复制算法，频繁的YGC又会导致大量对象的Copy，进而降低YGC效率，形成恶性循环。
  - **分配速率过大**：（内存增加速度 > 垃圾回收速度）可以观察出问题前后 Mutator 的分配速率，如果有明显波动可以尝试观察网卡流量、存储类中间件慢查询日志等信息，看是否有大量数据被加载到内存中。

- **策略**：

  - **增大Young区**，通常Old区设定为存活对象大小的2-3倍，剩下的空间可以预留给Young区。
  - 如果是分配速率问题：
    - **偶发较大**：通过内存分析工具找到问题代码，从业务逻辑上做一些优化。
    - **一直较大**：当前的 Collector 已经不满足 Mutator 的期望了，这种情况要么扩容 Mutator 的 VM，要么调整 GC 收集器类型或加大空间。



4. **场景六：单次 CMS Old GC 耗时长 \***

- **现象**：CMS GC 单次 STW 最大超过 1000ms，不会频繁发生，如下图所示最长达到了 8000ms。某些场景下会引起“雪崩效应”，这种场景非常危险，我们应该尽量避免出现。

![图片](https://raw.githubusercontent.com/PI-KA-CHU/Image-OSS/main/images640-20220622003941247.png)

- **原因**：
  - CMS 在回收的过程中，STW 的阶段主要是 *Init Mark* 和 *Final Remark* 这两个阶段，也是导致 CMS Old GC 最多的原因。
    - *Init Mark* ：从 GC Root 出发标记 Old 中的对象，处理完成后借助 BitMap 处理下 Young 区对 Old 区的引用，整个过程基本都比较快，很少会有较大的停顿。
    - *Final Remark*：Final Remark 的开始阶段与 Init Mark 处理的流程相同，但是后续多了 Card Table 遍历、Reference 实例的清理并将其加入到 Reference 维护的 `pend_list` 中，如果要收集元数据信息，还要清理 SystemDictionary、CodeCache、SymbolTable、StringTable 等组件中不再使用的资源。其中最容易出问题的是 ：
      - Reference 中的 FinalReference：主要观察 `java.lang.ref.Finalizer` 对象的 dominator tree，找到泄漏的来源。经常会出现问题的几个点有 Socket 的 `SocksSocketImpl` 、Jersey 的 `ClientRuntime`、MySQL 的 `ConnectionImpl` 等等
      - scrub symbol table 表示清理元数据符号引用耗时：符号引用是 Java 代码被编译成字节码时，方法在 JVM 中的表现形式，生命周期一般与 Class 一致，当 `_should_unload_classes` 被设置为 true 时在 `CMSCollector::refProcessingWork()` 中与 Class Unload、String Table 一起被处理。
- **策略**：由上述分析可知，问题主要来自Final Remark阶段，其中最容易出问题的是 【Reference 中的 FinalReference 和元数据信息处理中的 scrub symbol table 两个阶段】，通常不会大面积同时爆发，很多时候是单台的STW比较长，如果业务影响比较大，及时摘掉流量，具体后续优化策略如下：
  - FinalReference：找到内存来源后通过优化代码的方式来解决，如果短时间无法定位可以增加 `-XX:+ParallelRefProcEnabled` 对 Reference 进行并行处理。【在 Reference 类的问题处理方面，不管是 FinalReference，还是 SoftReference、WeakReference 核心的手段就是**找准时机 dump 快照**，然后用内存分析工具来分析。】
  - symbol table：观察 MetaSpace 区的历史使用峰值，以及每次 GC 前后的回收情况，一般没有使用动态类加载或者 DSL 处理等，MetaSpace 的使用率上不会有什么变化，这种情况可以通过 `-XX:-CMSClassUnloadingEnabled` 来避免 MetaSpace 的处理，JDK8 会默认开启 CMSClassUnloadingEnabled，这会使得 CMS 在 CMS-Remark 阶段尝试进行类的卸载。



4. **场景七：内存碎片&收集器退化**



5. **场景八：堆外内存 OOM**

- **现象**：内存使用率不断上升，甚至开始使用 SWAP 内存，同时可能出现 GC 时间飙升，线程被 Block 等现象，通过 top 命令发现 Java 进程的 RES 甚至超过了 -Xmx 的大小。出现这些现象时，基本可以确定是出现了堆外内存泄漏。

- **原因**：

  - 原因一：通过 `UnSafe#allocateMemory`，`ByteBuffer#allocateDirect` 主动申请了堆外内存而没有释放，常见于 NIO、Netty 等相关组件。
  - 原因二：代码中有通过 JNI 调用 Native Code 申请的内存没有释放。

- 策略：

  - 策略一：JVM 使用 `-XX:MaxDirectMemorySize=size` 参数来控制可申请的堆外内存的最大值。在 Java 8 中，如果未配置该参数，默认和 `-Xmx` 相等。可以通过 Debug 的方式确定使用堆外内存的地方是否正确执行了释放内存的代码。另外，需要检查 JVM 的参数是否有 `-XX:+DisableExplicitGC` 选项，如果有就去掉，因为该参数会使 System.gc 失效。（场景二：显式 GC 的去与留）

  - 策略二：Google perftools（统计调用 malloc 时进行内存分配的情况） + Btrace（对线上的Java调用栈进行追踪喝监控）工具进行分析，

![图片](https://raw.githubusercontent.com/PI-KA-CHU/Image-OSS/main/images640-20220622224641446.png)



#### 基本的GC日志参数

- `XX:+HeapDumpOnOutOfMemoryError`：OOM时打印堆快照

![图片](https://raw.githubusercontent.com/PI-KA-CHU/Image-OSS/main/images640-20220622225239025.png)

&nbsp;

&nbsp;

#### OOM

- OOM场景：https://juejin.cn/post/6873299829784018952
- OOM优化及监控：https://juejin.cn/post/7074762489736478757#heading-1
- OOM实战案例：
  - Netty堆外内存泄露排查与总结：https://mp.weixin.qq.com/s?__biz=MjM5NjQ5MTI5OA==&mid=2651749037&idx=2&sn=d1d6b0348eea5cd80e2c7a56c8a61fa9&chksm=bd12a3e08a652af684fd8d96e81fc0e0fded69dd847051e6b0f791f3726da0415c9552ee2615&scene=21#wechat_redirect
  - Spring Boot内存泄露排查记：https://mp.weixin.qq.com/s?__biz=MjM5NjQ5MTI5OA==&mid=2651750037&idx=2&sn=847fb15d4413354355c33a46a7bccf55&chksm=bd12a7d88a652ecea5789073973abb9545e76a8972c843968a6efd1fb3a918ef07eed8abb37e&scene=21#wechat_redirect




## 参考

- ZGC：https://mp.weixin.qq.com/s/RFwXYdzeRkTG5uaebVoLQw
- GC：https://mp.weixin.qq.com/s/RFwXYdzeRkTG5uaebVoLQw