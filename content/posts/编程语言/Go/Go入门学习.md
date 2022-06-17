

+++

author = "pikachu"
title = "Go入门学习"
date = "2022-06-17"
description = " "
tags = [
    "go"
]
categories = [
    "it","go"
]

+++

&nbsp;



## Go学习资料

- Go语言高级编程：https://chai2010.cn/advanced-go-programming-book/ch1-basic/ch1-06-goroutine.html
- Go语言圣经：https://docs.hacknode.org/gopl-zh/index.html
- Go Web编程：https://learnku.com/docs/build-web-application-with-golang/about-this-book/3151





## Go常见面试题

> 通过面试题快速学习Go语言



#### Go和Java有什么不同

- https://juejin.cn/post/7041935236149542926

1. 性能上：golang的性能比Java更好，占用内存更少，使用goroutine避免内核态和用户态切换成本
2. 编译部署：Java通过虚拟机编译，使用JVM跨平台编译；Go中不存在虚拟机,针对不同的平台，编译对应的机器码
3. 功能上：
   - 接口：java等面向对象编程的接口是侵入式接口，需要明确声明自己实现了某个接口。 而Golang的非侵入式接口不需要通过任何关键字，只要一个类型实现了接口的所有方法，就是这个接口的实现。
   - 继承：Java的继承通过extends关键字完成，不支持多继承；Go语言的继承通过Struct的方式，子类只需要把基类作为成员放在子类的定义中，支持多继承。
   - 异常处理：java中错误（Error）和异常(Exception)被分类管理，golang中只有error，一旦发生错误逐层返回，直到被处理。
   - 并发：通常借助于共享内存（全局变量）作为线程间通信的媒介，通常会有线程不安全问题，使用了加锁（同步化）、使用原子类、使用volatile提升可见性等解决；但在Golang中使用的是通道（channel）作为协程间通信的媒介，多个goroutine之间通过Channel来通信，chan的读取和写入操作为原子操作，所以是安全的。
   - 垃圾回收和内存管理：https://www.163.com/dy/article/GGGCI2OT0518R7MO.html
     - 