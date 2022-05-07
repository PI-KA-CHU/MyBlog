+++
author = "pikachu"
title = "JUC包结构及AQS抽象队列同步器"
date = "2020-01-03"
description = " "
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

## 前言：Concurrent包的层次结构
> `java.util.concurrent`包下提供了大量针对并发编程的高性能、实用的工具类，其目录结构图如下：

![undefined](http://ww1.sinaimg.cn/mw690/0061iV1igy1gakyd35arej30b70b80sy.jpg)

> JUC包中包含了两个子包，分别是`atomic`（原子类）包和`lock`（可重入锁）包（**AQS就在lock包中**），其他还包括阻塞队列、`
executors`等，底层主要利用**CAS**和**volatile读写**实现，下面是`current`包的整体技术及功能实现图：

![undefined](http://ww1.sinaimg.cn/mw690/0061iV1igy1gakyhoj7aej30mu0dc0ss.jpg)

&nbsp;

## 一、AQS简介
> 同步器（AQS）是用来**构建锁和其他同步组件**的基础框架，负责同步状态的管理，线程的排队，等待和唤醒这些底层操作，主要依赖一个int成员变量来表示**同步状态**以及通过一个`FIFO`队列构成**等待队列**。它的子类必须**重写AQS的几个protected修饰的用来改变同步状态的方法**，其他方法主要实现了排队和阻塞机制。状态的更新使用`getState`，`setState`以及`compareAndSetState`这三个方法。

> 同步器**子类被推荐定义为自定义同步组件的静态内部类**，同步器提供了**独占式**和**共享式**获取同步状态的方法，可以方便不同类型的同步组件的使用，但是同步器本身没有实现任何同步接口，它定义了同步状态的获取和释放的逻辑，将同步状态的具体实现和释放交由自定义同步组件者实现，这是设计模式中的**模板方法设计模式**。

&nbsp;

## 二、AQS的模板方法设计模式
> AQS的设计是使用模板方法设计模式，将同步状态逻辑封装在AQS（如`acquire`和`release`）中，将同步状态的具体实现由开发者实现（如`tryAcquire`和`tryRelease`），开发者可以自主实现同步资源的获取，如采用公平或者非公平方式获取，下面简单举个例子

#### AQS中的方法
```
// 由开发者实现
protected boolean tryAcquire(int arg) {
        throw new UnsupportedOperationException();
}

// AQS封装的执行逻辑，调用开发者实现的方法
public final void acquire(int arg) {
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
 }

```

#### ReentrantLock中NonfairSync（继承AQS）重写的方法
```
protected final boolean tryAcquire(int acquires) {
	// 继承者重写了tryAcquire，提供了具体实现（省略）
    return nonfairTryAcquire(acquires);
}
```

#### AQS中可重写的方法
![undefined](http://ww1.sinaimg.cn/mw690/0061iV1igy1gakmixncwbj30s60btwf3.jpg)

#### AQS提供的模板方法
![undefined](http://ww1.sinaimg.cn/mw690/0061iV1igy1gakmlh8gxtj30s80gxgn1.jpg)

&nbsp;

## 三、AQS的构成
- 下图为AQS的核心构成，AQS提供了**共享式锁**和**独享式锁**接口，并且封装了下面实线方框的相关接口，虚线方框则是由开发者（锁开发者）具体实现的方法，比如锁的获取是否公平（公平式抢占和非公平式抢占）等。
![image.png](http://ww1.sinaimg.cn/mw690/0061iV1igy1gakmtk1ml6j30ma0b574w.jpg)

#### 同步方式
- **独占式锁**
	- `void acquire(int arg)`：独占式获取同步状态，如果获取失败则插入同步队列进行等待；
	- `void acquireInterruptibly(int arg)`：与acquire方法相同，但在同步队列中进行等待的时候可以检测中断；
	- `boolean tryAcquireNanos(int arg, long nanosTimeout)`：在`acquireInterruptibly`基础上增加了超时等待功能，在超时时间内没有获得同步状态返回false;
	- `boolean release(int arg)`：释放同步状态，该方法会唤醒在同步队列中的下一个节点
- **共享式锁**
	- `void acquireShared(int arg)`：共享式获取同步状态，与独占式的区别在于同一时刻有多个线程获取同步状态；
	- `void acquireSharedInterruptibly(int arg)`：在`acquireShared`方法基础上增加了能响应中断的功能；
	- `boolean tryAcquireSharedNanos(int arg, long nanosTimeout)`：在`acquireSharedInterruptibly`基础上增加了超时等待的功能；
	- `boolean releaseShared(int arg)`：共享式释放同步状态，共享锁被多个线程读时锁会被多个线程占有

#### 数据结构
- **同步队列（Node结点：head、tail）**
	- `volatile int waitStatus`：节点状态
	- `volatile Node prev`：当前节点/线程的前驱节点
	- `volatile Node next`：当前节点/线程的后继节点
	- `volatile Thread thread`：加入同步队列的线程引用
	- `Node nextWaiter`：等待队列中的下一个节点
	- **节点状态**
		- `int CANCELLED = 1`：节点从同步队列中取消
		- `int SIGNAL = -1`：后继节点的线程处于等待状态，如果当前节点释放同步状态会通知后继节点，使得后继节点的线程能够运行；
		- `int CONDITION = -2`：当前节点进入等待队列中
		- `int PROPAGATE = -3`：表示下一次共享式同步状态获取将会无条件传播下去
		- `int INITIAL = 0`：初始状态
- **线程状态**：`state`
- **锁拥有者**：`exclusiveOwnerThread`

&nbsp;

## 四、参考
- https://juejin.im/post/5aeb055b6fb9a07abf725c8c
- https://juejin.im/post/5aeb07ab6fb9a07ac36350c8