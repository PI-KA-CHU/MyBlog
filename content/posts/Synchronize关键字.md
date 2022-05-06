+++
author = "pikachu"
title = "Synchronize关键字"
date = "2019-12-17"
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

## 一、初识Synchronize

#### what

> Synchronize是JVM的底层关键字，可以为对象的方法或者代码块加锁，加锁的代码同一时刻最多只能有一个线程执行。多个线程并发访问同一个对象的加锁同步代码时，一个时间内只能有一个线程获取到锁，其他线程会进入等待队列，待锁被释放后再进行抢占。Synchronize修饰的方法和代码块具有同步性和可见性。

#### where和why

> Synchronize被使用在多线程并发的场景，多个线程并发对一块非线程安全的代码进行操作，会出现线程安全问题，出现无法预料的结果。对被并发执行的代码添加Synchronize关键字，可以将多线程执行的代码同步化，同一时间内只能有一个线程获取到锁并进行操作，可以保证线程的变化（共享数据的变化）被其他线程看到，此时的代码块或方法是线程安全的。

#### how

- 修饰实例方法（对象锁）
```
public synchronized void synMethod(){
	//方法体
}
```

- 修饰静态方法（类锁）
```
public static synchronized void synMethod(){
	//方法体
}
```

- 修饰代码块
```
public Object synMethod(Object a1){
    synchronized(a1){
		//一次只能有一个线程进入
    }
}
```

&nbsp;

## 二、Synchronize的底层原理
- HotSpot JVM： https://www.cs.princeton.edu/picasso/mats/HotspotOverview.pdf
- 参考： https://blog.csdn.net/javazejian/article/details/72828483

> Synchronize主要用于修饰方法和代码块，两者在JVM中的底层实现是不同的，同步代码块的实现主要基于对象监视器（Monitor）实现，其主要是利用`monitorenter`和`monitorexit`指令实现代码同步；同步方法则是依靠方法修饰符上的`ACC_SYNCHRONIZED`标识实现同步。两者的JVM指令（利用javap命令对.class文件进行反编译）如下图所示：

**同步代码块**：

![image.png](http://ww1.sinaimg.cn/mw690/0061iV1igy1g9zkuemrbbj313y0jodjj.jpg)

**同步方法**：

![image.png](http://ww1.sinaimg.cn/mw690/0061iV1igy1g9zkw839yhj30mp07kt97.jpg)

#### 2.1 对象的同步属性
> Synchronize的底层同步（**重量级锁**）就是利用Monitor监视器实现线程间的互斥，进而达到同步的效果，下面简单描述Java对象在内存中的存储结构及Monitor如何实现同步监视。

##### 2.1.1 对象的内存布局
- **对象头（Header）**
	- **Mark Word**：主要存储**对象运行时数据**，如存储**哈希码、GC分代年龄、锁状态标志、线程持有的锁、偏向线程ID、偏向时间戳等**，长度在32位和63位的虚拟机中分别为32bit和64bit。Mark Word被设计为非固定大小的数据结构，在不同的锁状态下存储不同的内容，其中偏向锁是Java 6之后对`synchronize`的优化新添加的，锁标识位是01；重量级锁的锁标识位是10，其中指针指向monitor对象的起始地址，monitor在下面会进行较详细的介绍。
	![image.png](http://ww1.sinaimg.cn/mw690/0061iV1igy1ga05ligouxj30iy091wfn.jpg)
		- 例：在对象未被锁定的状态下，32bit的Mark Word中的25bit用于存储对象哈希码，4bit用于存储对象分代年龄，2bit用于存储锁标志位，1bit固定为0（当1时候表示偏向锁）。
	
	- **类型指针**:即对象指向它的**类元数据的指针**，虚拟机通过这个指针来确定这个对象是哪个类的实例。如果是数组对象还会存储数组长度的数据。
- **实例数据（Instance Data）**：对象真正存储的有效信息，即程序代码中所定义的各种类型的字段内容（包括父类字段）。
- **对齐填充（Padding）**：非必要存在，占位符（对象的大小必须是8的整数倍，不足时补充对齐）。

##### 2.2.1 monitor监视器底层
> 每个对象都存在对应的monitor与之关联，对象与其`monitor`之间的关系有存在多种实现方式，如monitor可以与对象一起创建销毁或当线程试图获取对象锁时自动生成，monitor对象被某个线程持有后会进入锁定状态，monitor是由`ObjectMonitor`实现，其数据结构如下（C++源码）：
```
ObjectMonitor() {
    _header       = NULL;
    _count        = 0; //记录线程获取锁的次数个数
    _waiters      = 0,
    _recursions   = 0; //线程的重入次数，为0时释放锁
    _object       = NULL;
    _owner        = NULL; //持有ObjectMonitor对象（锁）的线程，释放锁时为NULL
    _WaitSet      = NULL; //处于wait状态的线程，会被加入到_WaitSet（如线程被调用wait方法后，需要等待notify方法唤醒）
    _WaitSetLock  = 0 ;
    _Responsible  = NULL ;
    _succ         = NULL ;
    _cxq          = NULL ;
    FreeNext      = NULL ;
    _EntryList    = NULL ; //存放处于等待锁阻塞（block）状态的线程
    _SpinFreq     = 0 ;
    _SpinClock    = 0 ;
    OwnerIsThread = 0 ;
  }
```
上面为monitor对象的数据结构，主要包括`_EntryList`和`_WaitSet`队列及`_owner`锁持有者和其他的信息，获取到`monitor`对象（锁）的线程可以正常执行代码。

![image.png](http://ww1.sinaimg.cn/mw690/0061iV1igy1ga06nrkkjlj30et08vgn0.jpg)

#### 2.2 synchronize锁状态的变化

> synchronize在1.6版本后进行了优化，获取锁的过程为**偏向锁**（乐观锁，适应于一个线程执行的场景，进行简单的线程Id校验，减少轻量级锁的CAS次数） - **轻量级锁**（乐观锁，适用于**多线程交替执行同步块的场景**，轻量级锁认为当前线程的竞争程度很轻，即可能是两个线程交互执行，只要稍微（自旋）等待一会就可以获取到锁，如果自旋次数超过一定次数，则认为当前锁竞争激烈，升级为重量级锁） - **重量级锁**（悲观锁，适用于**多线程同时进入同步块场景**，需要获取monitor对象监视器，阻塞除了拥有锁以外的其他线程）

![image.png](http://ww1.sinaimg.cn/mw690/0061iV1igy1ga0zhgwfq8j30wy0fktee.jpg)

- **偏向锁加锁过程**
	- 1. 访问`Mark Work`中偏向锁的标识是否为1（默认为1，即开启状态）
	- 2. 如果为可偏向状态，则检查`_owner`线程Id是否指向当前线程，如果是，进入步骤5，否则进入步骤3
	- 3. 如果线程ID未指向当前线程，则通过`CAS`操作竞争锁。如果加锁成功（成功加锁后获取锁的线程再次进入时只需校验是否当前获取锁的线程Id，无需再进行CAS操作），则执行步骤5；如果加锁失败，则执行步骤4。
	- 4. 如果`CAS`获取偏向锁失败，则表示当前存在竞争，在到达全局安全点时，获取到偏向锁的线程被挂起，**偏向锁升级为轻量级锁**，然后被阻塞在安全点的线程继续执行同步代码。
	- 5. 执行同步代码。
	- 6. **释放锁**：偏向锁不会由线程主动释放，而是等到其他竞争锁的线程出现时，暂停拥有偏向锁的线程，检查线程是否存活，如果不存活，则恢复到无锁状态，允许其他线程竞争；如果存活，则挂起持有偏向锁的线程，将对象头`Mark word`修改为指向锁记录指针的标识，**锁升级为轻量级锁状态（00）**，最后重新唤醒挂起的线程。

- **轻量级锁加锁过程**
	- 参考： 
		- https://gorden5566.com/post/1019.html
		- https://www.cnblogs.com/paddix/p/5405678.html
		- https://www.zhihu.com/question/53826114
	- 1. 在代码进入同步块的时候，如果同步对象锁状态为无锁状态（锁标志位为“01”状态，是否为偏向锁标识为“0”），虚拟机首先将在**当前线程的栈帧**中建立一个名为**锁记录**（`Lock Record`）的空间，用于存储锁对象目前的`Mark Word`的拷贝，官方称之为`Displaced Mark Word`。  
	![image.png](http://ww1.sinaimg.cn/mw690/0061iV1igy1ga0yp11ok4j30t00dqtd3.jpg)
	- 2. 拷贝对象头中的`Mark Word`复制到锁记录中。
	- 3. 拷贝成功后，虚拟机将使用`CAS`操作尝试将对象的`Mark Word`更新为指向`Lock Record`的指针，并将`Lock record`里的`owner`指针指向`object mark word`。如果更新成功，则执行步骤4，否则执行步骤5。
	- 4. 如果这个更新动作成功了，那么这个线程就拥有了该对象的锁，并且对象`Mark Word`的锁标志位设置为“00”，即表示此对象处于**轻量级锁定状态**，这时候线程堆栈与对象头的状态如下图所示。  
	![image.png](http://ww1.sinaimg.cn/mw690/0061iV1igy1ga0yurrfbqj30sa0iyag8.jpg)
	- 5. 如果这个更新操作失败了，虚拟机首先会检查对象的`Mark Word`是否指向当前线程的栈帧，如果是就说明当前线程已经拥有了这个对象的锁（**重入**，每次获取轻量级锁时都会创建一个 `Lock Record`，锁重入时会创建多个指向同一个`Object`的`Lock Record`，除第一次设置`Displaced Mark Word` ，后面均设置为 null），那就可以直接进入同步块继续执行。否则说明**多个线程竞争锁**，（多次自旋无果后）轻量级锁就要**膨胀为重量级锁**，锁标志的状态值变为“10”，`Mark Word`中存储的就是指向重量级锁（互斥量）的指针，后面等待锁的线程也要进入**阻塞状态**。 而当前线程便尝试使用自旋来获取锁，自旋就是为了不让线程阻塞，而采用循环去获取锁的过程。
	- 6. **释放锁**：利用`CAS`操作把当前线程的栈帧中的`Displaced Mark Word`替换回锁对象的`Mark Word`中去，如果替换成功，则解锁成功，恢复到无锁的状态（01）。若替换失败，则轻量级锁膨胀为重量级锁后再解锁。

- **重量级锁**

#### Synchronize的优化

- **锁消除**
	- jitwatch下载地址： https://github.com/AdoptOpenJDK/jitwatch
	- 参考博客： 
		- http://www.cnblogs.com/stevenczp/p/7975776.html
		- https://www.cnblogs.com/stevenczp/p/7978554.html
> 例如StringBuffer是一个线程安全的类，其内部方法使用Synchronize修饰，但是如果我们是在单线程下使用StringBuffer进行字符串操作，那么不存在线程安全问题（线程封闭），但是频繁的加锁和解锁会造成性能消耗，此时Java在运行时会把锁消除进行优化（运行时优化，javap无法查看），可以利用工具查看Java最终运行的汇编代码，利用汇编指令表可以对照查看。

- **锁粗化**
> 例如在for循环中循环的加锁，会频繁的进行加锁解锁操作，造成性能消耗，Java会进行优化，将锁范围进行扩大，例如扩到到for循环外部。

- **偏向锁**
（参考上文）

- **轻量级锁**
（参考上文）

- **自旋锁**
> 轻量级锁失败后，虚拟机为了避免线程真实地在操作系统层面挂起，还会进行一项称为自旋锁的优化手段。这是基于在大多数情况下，线程持有锁的时间都不会太长，如果直接挂起操作系统层面的线程可能会得不偿失，毕竟操作系统实现线程之间的切换时需要从用户态转换到核心态，这个状态之间的转换需要相对比较长的时间，时间成本相对较高，因此自旋锁会假设在不久将来，当前的线程可以获得锁，因此虚拟机会让当前想要获取锁的线程做几个空循环(这也是称为自旋的原因)，一般不会太久，可能是50个循环或100循环，在经过若干次循环后，如果得到锁，就顺利进入临界区。如果还不能获得锁，那就会将线程在操作系统层面挂起，这就是自旋锁的优化方式，这种方式确实也是可以提升效率的。最后没办法也就只能升级为重量级锁了。

&nbsp;

## 三、总结

> 一个`synchronize`就涉及到大量的知识点，文章主要根据网易微专业视频并查找相关的博客、书籍学习后总结而成，主要涉及到`synchronize`锁状态及底层同步处理的过程，如有错误或问题，请联系笔者。
