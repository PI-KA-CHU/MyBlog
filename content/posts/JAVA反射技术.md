+++
author = "pikachu"
title = "JAVA反射技术"
date = "2018-12-11"
description = "Lorem Ipsum Dolor Si Amet"
tags = [
    "java"
]
categories = [
    "IT",
]
+++



**参考**：
- https://www.jianshu.com/p/104aa6e2342d
- https://www.jianshu.com/p/2315dda64ad2

#### Java反射的原理
- Java反射机制可以让我们<b>在编译期（Compile Time）之外的运行期（Runtime）获得任何一个类的字节码。包括接口、变量、方法等信息。</b>还可以让我们在运行期实例化对象，通过调用get/set方法获取变量的值。


#### Java反射的功能：

- 可以判断运行时对象所属的**类**
- 可以判断运行时对象所具有的**成员变量和方法**
- 通过反射甚至可以**调用private的方法**
- **生成动态代理**
- Java反射的功能，一句话总结就是：**反射用于在运行时检测和修改某个对象的结构及其行为。**
&nbsp;

#### 实现Java反射的类

- Class：它表示正在运行的Java应用程序中的类和接口
- Field：提供有关类或接口的属性信息，以及对它的动态访问权限
- Constructor：提供关于类的单个构造方法的信息以及对它的访问权限
- Method：提供关于类或接口中某个方法信息
- 注：**Class类是Java反射中最重要的一个功能类，所有获取对象的信息(包括：方法/属性/构造方法/访问权限)都需要它来实现**

&nbsp;
