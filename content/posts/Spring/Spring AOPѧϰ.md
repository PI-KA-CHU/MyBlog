+++
author = "pikachu"
title = "Spring AOP学习"
date = "2019-07-10"
description = ""
tags = [
	"",
    "spring"
]

categories = [
    "it", "spring"

]

+++


## 一、什么是AOP？

#### 1.1、AOP的定义

- AOP即`面向切面编程`，相比OOP —— 面向对象编程（最基本单位为`类和实例`），AOP的基本单位为`切面`。AOP编程主要可以用在如：`权限校验`、`日志服务`、`事务控制`、`异常处理`等，能够对业务进行统一管理编程，`减少了代码的冗余`，`降低了业务代码也服务管理部分的耦合程度`，同时`提高了代码的可维护性`，对下面是Spring中关于AOP的定义：

> 面向切面——Spring提供了面向切面编程的丰富支持，允许通过分离应用的`业务逻辑与系统级服务`（例如审计（auditing）和事务（transaction）管理）进行内聚性的开发。应用对象只实现它们应该做的——完成业务逻辑——仅此而已。它们并不负责（甚至是意识）其它的系统级关注点，例如日志或事务支持。


#### 1.2、AOP的的基本概念

- `通知/增强（Adivce） = 定位到方法后做什么事`
    - Before：前置通知（方法调用前）
    - After：后置通知（方法调用后，无论是否成功，相当于`Finally`代码块）
    - After-returning：最终通知（方法调用成功后）
    - After-throwing：异常通知（在方法抛出异常后调用通知）
    - Around：环绕通知（可以在目标方法前或方法后执行，即可以在环绕方法里调用目标方法）
    ```
    @Around("execution(...)")
    public Object around(ProceedingJoinPoint joinPoint) throws Throwable {
        System.out.println("我是环绕通知前....");
        //执行目标函数
        Object obj= (Object) joinPoint.proceed();
        System.out.println("我是环绕通知后....");
        return obj;
    }
    
    ```
- `切点（Pointcut） = 定位到具体方法的表达式`
    - 切点可以认为是对应某个方法的点（可以被AOP扫描到的某个方法），它通常与通知一起使用，比如，不同方法的权限校验，使用AOP进行编程的话这几个方法属于不同的切点，多个切点则组成了切面。
        - `execution(* com.zdy..*(..))`：com.zdy包下所有类的所有方法.
        - `execution(* com.zdy.Dog.*(..))`: Dog类下的所有方法.
    
- `连接点（Join point）`
    - 连接点是一个比较虚拟的概念，可以理解为满足所有切点扫描的所有`时机`，每个连接点可以将切面代码插入到其中，并为之添加新的行为，如调用某个接口时，异常抛出时，或者修改某个字段时等。
- `切面（Aspect） = Advice + Pointcut`
    - 切面是切点和通知的集合，一般单独作为一个类进行管理。通知和切点共同定义了关于切面的全部内容，它在什么时候（Advice的时间）、什么地方（切点）完成什么功能（Advice的功能）。
- `引入（Introduction）`
    - 引用允许我们向现有的类添加新的方法或者属性
- `织入（Weaving）`
    - 组装方面来创建一个被通知对象。这可以在编译时完成（例如使用AspectJ编译器），也可以在运行时完成。Spring和其他纯Java AOP框架一样，在`运行时`完成织入。

&nbsp;
&nbsp;


## 二、Spring对AOP的支持

#### 2.1、Spring的代理模式

- AOP思想的实现一般都是基于`代理模式`，在JAVA中一般采用`JDK动态代理`，但是由于JDK动态代理只能代理接口，如果代理类的话就不行了。因此，Sping AOP会自动进行切换，因为Spring同时支持`JDK动态代理`、`CGLIB`和`AspectJ`，当对象有实现接口时，会默认采用JDK动态代理，否则使用CGLIB代理。
    - 如果目标对象的实现类`实现了接口`，Spring AOP 将会采用 `JDK 动态代理` 来生成 AOP 代理类；
        - 实现了接口如何强制使用CGLIB实现AOP？
         （1）添加CGLIB库，SPRING_HOME/cglib/*.jar
          （2）在spring配置文件中加入`<aop:aspectj-autoproxy proxy-target-class="true"/>`
    
    - 如果目标对象的实现类`没有实现接口`，Spring AOP 将会采用 `CGLIB` 来生成 AOP 代理类——不过这个选择过程对开发者完全透明、开发者也无需关心。


#### 2.2、CGLIB和JDK动态代理的区别

**a、原理区别**：

- JDK动态代理是利用`反射机制`生成一个实现代理接口的匿名类，在调用具体方法前调用`InvokeHandler`来处理。
- CGLIB是利用`asm开源包`，将代理对象类的class文件加载进来，通过修改其`字节码`生成子类来处理。

**b、功能区别**：

- JDK动态代理只能对`实现了接口的类`生成代理，而不能针对类。
- CGLIB是`针对类实现代理`，主要是对指定的类生成一个子类，覆盖其中的方法，因为是继承，所以该类或方法最好不要声明成final 

&nbsp;
&nbsp;

## 三、Sping AOP配置

**切面配置**：

- XML配置
```
<aop:config>
    <!-- 这是定义一个切面，切面是切点和通知的集合-->
    <aop:aspect id="do" ref="PermissionVerification">
    	<!-- 定义切点 ，后面是expression语言，表示包括该接口中定义的所有方法都会被执行-->
        <aop:pointcut id="point" expression="execution(* wokao666.club.aop.spring01.Subject.*(..))" />
        <!-- 定义通知 -->
        <aop:before method="canLogin" pointcut-ref="point" />
        <aop:after method="saveMessage" pointcut-ref="point" />
    </aop:aspect>
</aop:config>

```

- 注解配置（AspectJ注解）
    - AspectJ是一个AOP框架，它能在`编译期`对JAVA代码进行AOP编译，让其具有AOP功能，AspectJ是目前实现AOP框架中最成熟的，并且与JAVA完全兼容。AspectJ单独就是一门语言，它需要专门的编译器(ajc编译器)，Spring很机智回避了这点，转向采用动态代理技术的实现原理来构建Spring AOP的内部机制（`动态织入`），而AspectJ是`静态织入`，这是最根本的区别，并且Sping AOP`不尝试提供完整的AOP功能`，其注重的是Spring AOP与Spring IOC容器的结合，借助IOC优势处理切面问题。Spring AOP只是整合了AspectJ的那套注解，底层仍然是采用Spring AOP的`动态代理技术`实现。

**切面类**：
```
@Aspect //声明自己是一个切面类
public class MyAspect {
    /**
     * 前置通知
     */
     // @Before是增强中的方位
     // @Before括号中的就是切入点了
     // before()就是传说的增强(建言):说白了，就是要干啥事.
    @Before("execution(* com.zdy..*(..))")
    public void before(){
        System.out.println("前置通知....");
    }
}
```
**XML文件**：
```
//开启AspectJ功能.
<aop:aspectj-autoproxy />

<bean id="dog" class="com.zdy.Dog" />
<!-- 定义aspect类 -->
<bean name="myAspect" class="com.zdy.MyAspect"/>
```

&nbsp;
&nbsp;


#### 四、参考
- https://juejin.im/post/5aa7818af265da23844040c6
- https://www.cnblogs.com/leifei/p/8263448.html
- https://juejin.im/post/5a55af9e518825734d14813f