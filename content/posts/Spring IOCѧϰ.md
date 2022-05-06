+++
author = "pikachu"
title = "Spring IOC学习"
date = "2019-07-01"
description = "Lorem Ipsum Dolor Si Amet"
tags = [
	"java",
  "spring"
]
categories = [
    "IT"
]
+++



## 一、什么是IOC

- IOC（`控制反转`）又叫做DI（`依赖注入`），它描述了对象的定义和依赖的一个过程，也就是说，依赖的对象通过构造参数、工厂方法参数或属性注入，当对象实例后依赖的对象才被创建，当创建bean后容器注入这些依赖对象。与原来在类中使用其他类时相比（new Object()），这个过程是`反向`的，即对象与对象间的依赖不再是主动的去new 一个对象，而是交由Bean容器在对象实例化的时候进行注入。

&nbsp;

## 二、为什么要用IOC

> IOC是Spring的重要部分，那么为什么要使用IOC呢？下面是使用IOC与不使用的比较：


**a. 原始的对象间的关系**：

![image](https://user-images.githubusercontent.com/38284818/60417713-a9e24200-9c13-11e9-8674-e37098f470b5.png)

&nbsp;

**b.基于IOC容器的对象间的关系**：

![image](https://user-images.githubusercontent.com/38284818/60417734-ba92b800-9c13-11e9-82ca-f489a0b7fe79.png)

&nbsp;

> 从上面的图中对比可以看出，原始的对象间的关系存在着`耦合过高`的问题，在大型软件系统中其耦合程度更是极其复杂，IOC容器将复杂的系统分解成相互合作的对象，`降低了解决问题的复杂度`，并且由于Spring默认是单例模式，使得对象可以被灵活地`扩展和重用`，极大`降低了系统的开销`。通过IOC容器，实现了`具有依赖关系的对象间的解耦`。

&nbsp;

## 三、IOC的实现原理：

> IOC中最基本的技术就是`反射`，JAVA反射机制就是在`运行状态`中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能调用它的任意方法和属性；这种动态获取信息以及动态调用对象方法的功能称为JAVA语言的`反射机制`；这里简单讲述一下


#### 为什么要使用反射：
> 程序运行前需要先编译，编译过程中将代码中需要的类加载到JVM中，运行的时候进行内存分配（类的实例化），相当于加载的类已经是固定的，如果使用静态编译的话，增加某个类的创建需要重新编译整个软件，而使用反射机制（动态编译）则不需要，可以在运行的过程中动态的获取。

a. 无反射技术的工厂模式：
```
//构造工厂类
//也就是说以后如果我们在添加其他的实例的时候只需要修改工厂类就行了
class Factory{
     public static fruit getInstance(String fruitName){
         fruit f=null;
         if("Apple".equals(fruitName)){
             f=new Apple();
         }
         if("Orange".equals(fruitName)){
             f=new Orange();
         }
         return f;
     }
```
b. 基于反射技术的工厂模式：
```
class Factory{
    public static fruit getInstance(String ClassName){
        fruit f=null;
        try{
            f=(fruit)Class.forName(ClassName).newInstance();
        }catch (Exception e) {
            e.printStackTrace();
        }
        return f;
    }
}
```
> 由上面的代码示例可以看出，反射技术极大提升了代码的灵活性，由于无法知道需要创建的Bean类型，反射技术可以在运行时动态的调用构造方法进行类的动态创建，IOC实现的工厂模式即是使用反射技术，能否在运行时动态创建也是衡量一门语言是否是动态语言的标准。之一（可以查下动态语言和静态语言的区别）。

&nbsp;

#### 反射机制的作用：

- 在运行的时候能够判断任意对象所属的类
- 在运行时获取类的对象
- 在运行时访问java的属性、方法和构造方法等；

&nbsp;

#### 反射技术的优缺点：

- **优点**：能够动态的创建对象和编译，具有极强的灵活性。
- **缺点**：性能相对较差，使用反射基本是一种解释性操作。

&nbsp;


## 四、Spring IOC的重要内容

### 4.1 依赖注入的实现（DI）


#### 4.1.1 注入方式

- **基于`构造函数`的注入**

TextEditor.java
```
public class TextEditor {
   private SpellChecker spellChecker;
   public TextEditor(SpellChecker spellChecker) {
      System.out.println("Inside TextEditor constructor." );
      this.spellChecker = spellChecker;
   }
   public void spellCheck() {
      spellChecker.checkSpelling();
   }
}
```
SpellChecker.java
```
public class SpellChecker {
   public SpellChecker(){
      System.out.println("Inside SpellChecker constructor." );
   }
   public void checkSpelling() {
      System.out.println("Inside checkSpelling." );
   } 
}
```
MainApp.java
```
public class MainApp {
   public static void main(String[] args) {
      ApplicationContext context = 
             new ClassPathXmlApplicationContext("Beans.xml");
      TextEditor te = (TextEditor) context.getBean("textEditor");
      te.spellCheck();
   }
}
```
xml配置文件
```
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

   <!-- Definition for textEditor bean -->
   <bean id="textEditor" class="com.tutorialspoint.TextEditor">
      <constructor-arg ref="spellChecker"/>
   </bean>

   <!-- Definition for spellChecker bean -->
   <bean id="spellChecker" class="com.tutorialspoint.SpellChecker">
   </bean>

</beans>
```

&nbsp;

- **基于`setter方法`的注入**（由于注入是基于java`反射机制`实现的，即使没有 setter 声明的方法，也可以进行注入）

TextEditor.java
```
public class TextEditor{
   private SpellChecker spellChecker;
   // a setter method to inject the dependency.
   public void setSpellChecker(SpellChecker spellChecker) {
      System.out.println("Inside setSpellChecker." );
      this.spellChecker = spellChecker;
   }
   // a getter method to return spellChecker
   public SpellChecker getSpellChecker() {
      return spellChecker;
   }
   public void spellCheck() {
      spellChecker.checkSpelling();
   }
}
```
SpellChecker.java
```
public class SpellChecker {
   public SpellChecker(){
      System.out.println("Inside SpellChecker constructor." );
   }
   public void checkSpelling() {
      System.out.println("Inside checkSpelling." );
   }  
}
```
xml配置文件
```
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

   <!-- Definition for textEditor bean -->
   <bean id="textEditor" class="com.tutorialspoint.TextEditor">
      <property name="spellChecker" ref="spellChecker"/>
   </bean>

   <!-- Definition for spellChecker bean -->
   <bean id="spellChecker" class="com.tutorialspoint.SpellChecker">
   </bean>

</beans>
```

#### 4.1.2 注入配置
   
- 基于`XML`的注入配置（如上面的配置）
- 基于`注解`的注入配置（通过`@Autowired`进行注入）

&nbsp;

### 4.2 Spring IOC的初始化（Bean的初始化）

- 参考：https://www.jianshu.com/p/70886997c46b

- 主要分为三个步骤（容器的初始化是通过`refresh()`实现）
    - **定位**：通过`Resource`定位`BeanDefinition`，BeanDefinition定义了Bean的元信息、依赖关系等，即寻找Bean的过程。
    - **载入**：`BeanDefinition`的信息已经定位到了，第二步就是把定义的`BeanDefinition`在`Ioc容器`中转化成一个Spring内部标示的数据结构的过程。
    - **注册**：将抽象好的BeanDefinition统一`注册`到IoC容器中，IoC容器是通过`ConcurrentHashMap`来维护BeanDefinition信息的，key为beanName，value为BeanDefinition。


&nbsp;
&nbsp;

## 五、IOC的优缺点

**优点**：

- 实现了对象之间的`解耦`，基于单例模式可以有效减少系统资源的消耗。

**缺点**：

- 基于`反射`实现，性能会稍微差一些，单例模式引入了线程安全问题。
- 生成对象的步骤变得复杂了，需要投入学习成本和精力。


&nbsp;
&nbsp;

## 六、IOC的源码解析

![ClassPathXmlApplicationContext](https://user-images.githubusercontent.com/38284818/60594164-e611d000-9dd6-11e9-8255-bf4130cf4580.png)

**BeanFactory和ApplicationContext的区别**

- ApplicationContext接口`继承`自BeanFactory接口，同时继承了MessageSource和ResourceLoader等其他接口，相比BeanFactory，ApplicationContext提供了`更多的扩展功能`，如能够实现国际化访问、事务传播及AOP等服务。
- BeanFactory是`懒加载`（延迟加载），BeanFactory加载后，需要第一次调用`getBean()`方法才会实例化，而ApplicaitonContext实现的是`饿汉加载`，在容器初始化的时候，会实例化所有的bean。
- ApplicationContext可以`及时检查`Bean是否完成注入，BeanFactory需要调用getBean()的时候才会抛出异常。


&nbsp;
&nbsp;

#### 参考
- https://www.cnblogs.com/wang-meng/p/5597490.html
- https://blog.csdn.net/u010325193/article/details/80865672
- *https://blog.csdn.net/fuzhongmin05/article/details/61614873
- https://blog.csdn.net/java_gchsh/article/details/78111200
- https://blog.csdn.net/qq_19782019/article/details/85038081  （setter注入问题）
