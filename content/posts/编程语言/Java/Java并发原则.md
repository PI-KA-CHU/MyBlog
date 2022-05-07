+++
author = "pikachu"
title = "Java happens-before及as-if-serial原则"
date = "2019-12-30"
description = "Lorem Ipsum Dolor Si Amet"
draft = false
tags = [
    "Java",
    "多线程",
    "网易微专业"
]
categories = [
    "IT"
]
+++


## 一、happens-before
> 【happens-before定义了某些操作下的可见性和"有序性"】  
> 由于重排序（编译器重排和处理器重排）的底层规则的存在导致程序理解变得复杂，严重影响开发效率，为了解决此问题，JMM为程序员在上层提供了六条原则，这样我们就可以根据规则去推论跨线程的内存可见性问题，而不用再去理解底层重排序的规则。下面以两个方面来说。

#### 1.1 happens-before定义
> happens-before的概念最初由Leslie Lamport在其一篇影响深远的论文（《Time，Clocks and the Ordering of Events in a Distributed System》）中提出。JSR-133使用`happens-before`的概念来指定**两个操作之间的执行顺序**。由于这两个操作可以在一个线程之内，也可以是在不同线程之间。因此，**JMM可以通过happens-before关系向程序员提供跨线程的内存可见性保证**（如果A线程的写操作a与B线程的读操作b之间存在happens-before关系，尽管a操作和b操作在不同的线程中执行，但JMM向程序员保证a操作将对b操作可见）。具体的定义为：

- 如果一个操作`happens-before`另一个操作，那么第一个操作的执行结果将对第二个操作可见，而且第一个操作的执行顺序排在第二个操作之前。
	- 此规则是**JMM对程序员的承诺**。从程序员的角度理解`happens-before`：如果A happens-bofore B，那么Java内存模型将向程序员保证——A操作的结果将对B可见，且A的执行顺序排在B之前。真正的运行顺序可能是不一样的，但是运行结果可以这么认为。
- 两个操作之间存在`happens-before`关系，并不意味着Java平台的具体实现必须要按照`happens-before`关系指定的顺序来执行。如果重排序之后的执行结果，与按`happens-before`关系来执行的结果一致，那么这种重排序并不非法（也就是说，JMM允许这种重排序）。
	- 此规则是**JMM对编译器和处理器重排序的约束规则**。JMM是遵循一个基本原则：只要不改变程序的执行结果（指单线程和正确同步的多线程），编译器和处理器怎么优化都行。	JMM这么做的原因是：程序员对于这两个操作是否真的被重排序并不关心，关心的是程序执行时的语义不能被改变（即执行结果不能被改变）。其本质上和`as-if-serial`类似。


#### 1.2 JMM原生happens-before规则
> 下面规则为**Java语言中无需任何同步手段保障就能成立的先行发生的规则**。
- **程序次序规则**：在一个线程内，按照程序代码顺序，书写在前面的操作`Happens-Before`书写在后面的操作（前面操作对后面代码可见）
- **管程锁定规则**：一个`unlock`操作`Happens-Before`后面对同一个锁的`lock`操作。
- **volatile变量规则**：一个线程对`volatile`变量的写入操作`Happens-Before`另外线程对这个变量的读操作。（volatile写操作对后续读可见）
- **线程启动规则**：Thread对象的`start()`方法`Happens-Before`此线程的每一个动作。
- **线程中断规则**： 对线程`interrupt()`方法的调用`Happens-Before`被中断线程的代码检测到中断事件的发生，可以通过`Thread.interrupt()`方法检测到是否有中断发生。
- **线程终止规则**：线程中的所有操作都`Happens-Before`对此线程的终止检测。
- **对象终结规则**：一个对象的**初始化完成**（构造方法执行结束）`happens-before`它的`finalize()`方法的开始。
- **传递性**：如果某个动作a `happens-before` 动作b，且b `happens-before` 动作c，则有a `happens-before` c。


#### 1.3 happens-before推导
> Java中原生满足`happens-before`关系的有以上八条，下面是由上面八条推导的规则，如：
- 将一个元素放入一个线程安全的队列的操作`Happens-Before`从队列中取出这个元素的操作
- 将一个元素放入一个线程安全容器的操作`Happens-Before`从容器中取出这个元素的操作
- 在CountDownLatch上的倒数操作`Happens-Before`CountDownLatch#await()操作
- 释放Semaphore许可的操作`Happens-Before`获得许可操作
- Future表示的任务的所有操作`Happens-Before`Future#get()操作
- 向Executor提交一个Runnable或Callable的操作`Happens-Before`任务开始执行操作


#### 1.4 小结
- 如果两个操作不存在上述（前面8条 + 后面6条）任一一个`happens-before`规则，那么这两个操作就没有顺序的保障，JVM可以对这两个操作进行重排序。如果操作A `happens-before`操作B，那么操作A在内存上所做的操作对操作B都是可见的。
- `happen-before`原则是JMM中非常重要的原则，它是判断**数据是否存在竞争**、**线程是否安全**的主要依据，**保证了多线程环境下的可见性**。
- 具体的例子可以参考： https://www.cnblogs.com/chenssy/p/6393321.html

&nbsp;

## 二、as-if-serial
> **as-if-serial**的语义是：**不管怎么重排序（编译器和处理器为了提高并行度），（单线程）程序的执行结果不能被改变**。编译器、处理器都必须遵守`as-if-serial`语义。为了遵守 as-if-serial 语义，编译器和处理器不会对存在**数据依赖关系**的操作做重排序，因为这种重排序会改变执行结果。但是，如果操作之间不存在数据依赖关系，这些操作就可能被编译器和处理器重排序，从而提高处理性能。

&nbsp;

## 三、参考
- https://www.cnblogs.com/chenssy/p/6393321.html
- https://juejin.im/post/5ae6d309518825673123fd0e
- http://ifeve.com/from-singleton-happens-before/