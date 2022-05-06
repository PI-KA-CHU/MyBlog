+++
author = "pikachu"
title = "Volatile关键字"
date = "2019-12-19"
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


## 一、初识Volatile

#### what
> `volatile`是Java语言提供的一种**较弱的同步机制**，用来确保将变量的更新操作通知到其他线程，`volatile`修饰的变量具有**可见性**和**有序性**，即变量的修改不会被缓存在寄存器或者其他高速缓存器中（可见），并且不会被遍历器和处理器**指令重排**（有序）。`synchronize`提供了**原子性**和**可见性**，与`synchronize`相比，`volatile`不保证原子性，所以在多线程并发情况下仍可能会出现线程安全问题，另外两者的**有序性**也是不一样的，`volatile`的有序性是禁止指令重排，而`synchronize`的有序性是建立在同步之上的，就是说保证单线程执行，并且同步区域的变量会被写入主存（对其他线程可见），但是同步区域仍然可以进行指令重排，这也就是DCL（双重检验锁）的问题。

#### where and why
> `volatile`主要应用在需要**确保自身状态可见**和**指令重排会出现问题**的场景，如确保所引用对象的状态可见性、标识一些重要的程序生命周期事件的发生（如初始化或关闭）。`volatile`没有加锁的消耗，因此在一些不需要加锁访问并且需要对其他线程可见的场景能提供更好的性能。
`volatile`的使用条件：
> - 对变量的写入操作不依赖变量的当前值，获取保证只有**单线程**更新变量的值。
> - 该变量不会与其他状态变量一起纳入不变性条件中。
> - 在访问变量时**不需要加锁**。

#### how
> `volatile`用于修饰共享变量，保证变量的修改在多线程间的可见性和有序性（禁止指令重排），下面的例子是**单例模式**中的双重检验锁（DCL）模式，使用`volatile`保证要创建的对象的可见性，否则在同步代码块中，对象的`new`在底层分为多步骤进行，如：a.分配内存、b.初始化对象、c.对象引用指向内存区域，由于同步代码块中可以进行**指令重排**（由abc变成acb，可参考笔者的另一篇博客`Java并发基础`），其他线程在判断对象不为空时对象可能还没初始化，获取到的是空的对象，由此造成安全问题。
```
public class Singleton {

    private static volatile Singleton singleton;

    private Singleton(){

    }

    public static Singleton getInstance(){
        if(singleton != null){
            synchronized (Singleton.class){
                if (singleton != null){
                    // 可能会有指令重排
                    singleton = new Singleton();
                }
            }
        }
        return singleton;
    }

}
```

&nbsp;

## 二、Volatile的底层原理
> JMM（Java内存模型）定义了volatile的内存语义，当一个变量声明为volatile时，它的读写操作将具有特殊的含义。

#### 2.1 volatile内存语义

##### 2.1.1 可见性的内存语义
- **写的内存语义**：当写一个volatile变量时，JMM会把该线程对应的本地内存中的共享变量值刷新到主内存。其他线程在监听到总线对该内存地址的写入后，如果他们的对该地址内存的缓存状态为S，则让缓存失效并置为I。
- **读的内存语义**：当读一个volatile变量时，由于volatile写入时`MESI`协议会把该线程对应的本地内存置为无效，线程接下来从主内存中读取共享变量。

##### 2.1.2 可见性的内存语义实现
- 如果对声明了volatile的变量进行写操作，JVM就会向处理器发送一条`Lock`前缀的指令，该指令将变量所在缓存行的数据写回系统内存。但是即使内存被写回，缓存在其他处理器上的数据仍然是旧数据，在多核处理器下，为了保证各个处理器的缓存是一致的，CPU厂商制定了**缓存一致性协议**，**每个处理器通过嗅探在总线上传播的数据来检查自己缓存的值是不是过期了**，当处理器发现自己缓存行的内存地址被修改，就会将当前处理器的缓存行设置为无效（仅仅设置为无效，不会直接更新），当需要数据的时候发现缓存行状态为无效，则从主存中读取。

##### 2.2.1 有序性的内存语义
- **volatile读**：volatile读之后的操作不会被重排序到volatile读之前
- **volatile写**：volatile写之前的操作不会被重排到volatile之后
- **先volatile写-后volatile读**：不可重排序

##### 2.2.2 有序性的内存语义实现

> 通过**对编译器和处理器的重排序的限制**，从而实现了volatile的内存语义。
- **对编译器重排序的限制**
	- 为了实现`volatile`的内存语义，在编译器生成字节码时，会在指令序列中插入内存屏障来禁止特定类型的**处理器排序**。
![image.png](http://ww1.sinaimg.cn/large/0061iV1igy1ga30gtwmkzj30ou0giab4.jpg)
- **对处理器重排序的限制**
![image.png](http://ww1.sinaimg.cn/large/0061iV1igy1ga30hpgonvj30p00jawgg.jpg)
	- 对于编译器来说，发现一个最优布置来最小化插入屏障的总数几乎是不可能的，为此，JMM采取了保守策略（即**在volatile写的前面和后面分别插入内存屏障，在volatile读操作后面插入两个内存屏障**）：
		- 1. 在在每个volatile写操作的前面插入一个StoreStore屏障；
		- 2. 在每个volatile写操作的后面插入一个StoreLoad屏障；
		- 3. 在每个volatile读操作的后面插入一个LoadLoad屏障；
		- 4. 在每个volatile读操作的后面插入一个LoadStore屏障。
	- 屏障的作用：
		- **StoreStore屏障**：禁止上面的普通写和下面的volatile写重排序；
		- **StoreLoad屏障**：防止上面的volatile写与下面可能有的volatile读/写重排序
		- **LoadLoad屏障**：禁止下面所有的普通读操作和上面的volatile读重排序
		- **LoadStore屏障**：禁止下面所有的普通写操作和上面的volatile读重排序
	- 《Java并发编程艺术》书中例图

	![image.png](http://ww1.sinaimg.cn/mw690/0061iV1igy1ga5vosa32cj30h80anq4c.jpg)
	![image.png](http://ww1.sinaimg.cn/mw690/0061iV1igy1ga5vprufguj30h809vgmv.jpg)

&nbsp;

## 三、总结

> 扩展：虚拟机规范中，写64位的double和long分成了两次32位值的操作（非原子性），而添加了`volatile`修饰的long和double读写总是**原子的**，读写引用也是原子的。商业JVM则不会有此问题，考虑到实际应用都实现了原子性。
> 本篇博客对volatile的可见性和有序性进行了描述，并针对底层实现进行了简单总结，设计到编译器、处理器的指令重排机制及实现volatile可见性的**MESI**缓存一致性协议、**编译器Locl前缀的指令**和实现有序性的**CPU内存屏障**，内容主要总结自其他大佬博客及个人理解，有不正确的地方希望大家指出。

&nbsp;

**参考**
- https://juejin.im/post/5ae9b41b518825670b33e6c4#heading-1
- https://gorden5566.com/post/1018
- https://blog.csdn.net/tb3039450/article/details/67636391

&nbsp;



