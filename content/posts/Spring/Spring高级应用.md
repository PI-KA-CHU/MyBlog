+++
author = "pikachu"
title = "Spring高级应用"
date = "2019-12-07"
description = "Lorem Ipsum Dolor Si Amet"
draft = false
tags = [
	"java",
	"spring"
]

categories = [
    "IT", "Spring"

]

+++


## 一、如何基于Spring写优雅易扩展的代码


### 1.1 案例一

![image.png](http://ww1.sinaimg.cn/large/006H3ec5gy1g9ofobumg8j30vf0datco.jpg)

#### 1.1.1 如何实现订单金额的计算

- **策略模式**
实现面向接口编程，由于计算金额的方法是会增加的，比如促销方案是会变的，可以利用策略模式，将订单金额计算的方法抽象为接口，有不同的促销模式时候实现该接口（接口隔离原则），避免频繁的修改代码（开闭原则）

- **工厂模式**
	利用工厂方法实现方案计算方法实例的生成，（根据传入的String）利用反射获取各种金额计算方案的实现，如果是自己利用反射技术实现简单工厂的话，每次有新促销方案的话我们需要修改自定义的工厂，因为工厂需要初始化各个促销接口的实现类，为此引入下列几种方式，通过将变化的部分以配置文件的方式隔离（SPI）、Spring注入、Spring提供的工厂真正实现开闭原则。
	- **Java SPI**：`接口服务发现机制`，对接口的实现类自动完成加载。SPI机制可以避免硬编码的问题，厂商通过规定的配置规则，即`使用方提供规则，提供方根据规则把自己加载到提供方的思想`。在Java中通过`ServiceLoader`加载约定配置下的实现类，`ServiceLoader<Driver> s = ServiceLoader.load(Driver.class);`如：
		- 1. MySQL的驱动加载时，在`META/services`中以调用者给定的`完整接口名`命名文件，文件内容为该接口的`具体实现完整类名`；
		- 2. Spring中的`component-scan`
		注解标签，会将@Controller、@Service等的标签注入到容器中，也就是Spring制定了规则，开发者根据规则去实现。
		- 参考： https://zhuanlan.zhihu.com/p/28909673
		![image.png](http://ww1.sinaimg.cn/mw690/006H3ec5gy1g9qsftdrwmj30k007cdhh.jpg)
		![image.png](http://ww1.sinaimg.cn/mw690/006H3ec5gy1g9qk98mysdj30k00aujuf.jpg)

	- **Dubbo SPI**
	- **Spring实现**
		- `@Autowired Map<String,Order> map`，利用`Map可以自动将所有Order接口的实现类注入`
		- 利用`ApplicationContext`获取bean并实现方法的调用，即利用Spring提供的工厂模式

&nbsp;

## 二、如何提高工作效率

#### 为什么我们的工作效率不高

- 我们掌握的方法、技能、工具不多，不知道还有更好的开发模式，一直采用传统的堆代码开发，`要提高工作效率，需要多掌握更多更好的开发方法、工具和技能`。

#### 高薪人的优势是什么

- 能够解决更大的问题，更难的问题
- 相同时间解决的问题更多，绝对不是因为加班多才会薪资高

&nbsp;

## 三、如何基于Spring写解耦易扩展的代码

### 3.1 案例分析

#### 3.1.1 业务场景演示

**常规代码流程**：

![image.png](http://ww1.sinaimg.cn/mw690/006H3ec5gy1g9ogwyyd0jj30oa0630um.jpg)

**问题**
- 同步调用
- 如何邮件发送异常，则短信服务也无法进行，订票失败
- 违反单一原则（在订单方法中加入了邮件和短信通知）
- `与不是强相关的类耦合`（航班预定与邮箱、短信类） - 复用性极低
- 如果有新功能加入，如需要发微信、QQ，则会违反开闭原则，不易扩展

---

**基于`事件驱动`的流程（观察者模式）**：

![image.png](http://ww1.sinaimg.cn/mw690/006H3ec5gy1g9oh3kzm5fj30mw09vtbk.jpg)

**事件驱动**
- 发布者：OrderService类中的预定订单方法
- 事件：航班预定事件，即Order（订单）对象为事件
- 事件通道：spring（`ApplicationContext`提供事件的发布通道，如`publishEvent(Object o)`和`publishEvent(ApplicationEvent event)`方法，一个用于对象事件如订单生成Order对象、一个用于应用事件如应用启动关闭事件，可以用于Spring启动的时候进行相关的初始化任务）
- 监听者：邮件发送类、短信发送类

**Spring注解**
- `@Async`：异步执行注解
- `@Transactional`：添加事务
- 观察者模式
	
	- 监听者：`@EventListener`
	```
	@EventListener
	@Async //异步执行
	public void handleOrderEvent(Order order){
		//上面的传入对象参数即为监听的事件，该监听方法只对Order事件感兴趣
		Email mail = ...;
		this.sendMail(email);
	}
	```
	- 发布者：在发生订单事件的方法中调到Spring提供的`ApplicationContext`的发布方法
	```
	@Autowired
	ApplicationContext context;
	public void bookOrder(Order o){
		...
		context.publishEvent(order);
	}
	```

&nbsp;

## 四、如何基于Spring写异步、定时任务

**异步方式**
- 方式一：配置ApplicationEventMuticaster的Bean（基于事件的异步）
	- Spring提供，配置configuration的Bean，并在其中配置线程池（没有线程池配置则为同步）
	- 缺点：一刀切（所有的事件执行都变为异步的）
- 方式二：@EnableAsync @Async（基于Bean的异步）
	- 控制Bean的方法调用为异步执行（灵活），需要特定线程池则进行配置
	- `@EnableAsync`：开启SpringBoot异步模式
	- `@Async`：被标识的方法在调用的时候为异步调用，未被标识的方法为同步执行

**定时任务**
- `@EnableScheduling`：开启SpringBoot定时任务
- `@Schedule`：在执行逻辑的Bean（`@Component`）的方法上注释即可实现定时任务

&nbsp;

## 五、如何扩展我们的技术广度

> 针对事情快速应对，即想到相应的方法。需要有好的技术，想想如何达到高薪的位置，而不是单单想着拿高薪。
> 单机的技术使用跟分布式的使用是不同的，单机的可以充分利用Spring的高级用法，而分布式则更多的依赖于中间件，比如上面说的事件驱动编程，如果在Spring中可以利用其注解快速实现，而分布式环境中则需要利用`MQ`实现发布订阅模式，要能快速的想出解决方案，需要我们有足够的知识广度。

**扩展技术广度的方法**
- 参加课程

![image.png](http://ww1.sinaimg.cn/large/006H3ec5ly1g9qw191vpfj312k0lb422.jpg)

- 技术大牛带
- 项目的洗礼
- 去大的互联网公司
- 多看书、多交流探讨

&nbsp;

