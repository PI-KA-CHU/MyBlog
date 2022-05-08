+++
author = "pikachu"
title = "Spring Bean学习"
date = "2019-07-09"
description = " "
tags = [
	"",
    "spring"
]

categories = [
    "it", "spring"

]

+++


## 一、定义

#### bean的定义

- Bean是一个被`实例化`、`组装`、并被`Spring容器所管理`的对象，是由容器提供的`配置元数据`（如bean.xml）创建的。（下图是Bean与Spring容器的关系）

![image](https://user-images.githubusercontent.com/38284818/60699678-98f14380-9f27-11e9-8e33-f0edb056166c.png)

&nbsp;

#### bean的三种元数据配置（bean的创建）

- 参考： https://blog.csdn.net/isea533/article/details/78072133

1. 基于XML的配置
```
<beans>
    <bean name="" class=""></bean>
</beans>
```
2. 基于注解的配置
```
@Component
public class MyBeanConfig {

    @Autowired
    private Country country;

    @Bean
    public Country country(){
        return new Country();
    }

    @Bean
    public UserInfo userInfo(){
        return new UserInfo(country);
    }

}
```
3. 基于Java的配置（`@Configuration`实现自动依赖注入，且通过代理操作，而`@Component`则没有代理，且需要加上`@Autowired`，否则两个方法返回的是不同的实例）
```
@Configuration
public class MyBeanConfig {

    @Bean
    public Country country(){
        return new Country();
    }

    @Bean
    public UserInfo userInfo(){
        return new UserInfo(country());
    }

}
```


#### bean的作用域
（可通过Xml的`scope`属性及`@Scope`注解进行配置）

- singleton
- prototype
- request
- session
- global-session


#### bean的生命周期
- 转自： https://www.zhihu.com/question/38597960/answer/77600561
![image](https://user-images.githubusercontent.com/38284818/60722728-5c900880-9f64-11e9-8789-b0b574e5e29b.png)


- 1、Spring对Bean进行`实例化`（相当于程序中的new Xx()，`反射实现`）

- 2、Spring将值和Bean的`引用注入`进Bean对应的属性中（`反射实现`）
- 3、如果Bean实现了BeanNameAware接口，Spring将Bean的ID传递给`setBeanName()`方法（实现BeanNameAware清主要是为了通过Bean的引用来获得Bean的ID，一般业务中是很少有用到Bean的ID的）
- 4、如果Bean实现了BeanFactoryAware接口，Spring将`调用setBeanFactory`(BeanFactory bf)方法并把BeanFactory容器实例作为参数传入。（实现BeanFactoryAware 主要目的是为了获取Spring容器，如Bean通过Spring容器发布事件等）
- 5、如果Bean实现了ApplicationContextAwaer接口，Spring容器将`调用setApplicationContext`(ApplicationContext ctx)方法，把应用上下文作为参数传入.(作用与BeanFactory类似都是为了获取Spring容器，不同的是Spring容器在调用setApplicationContext方法时会把它自己作为setApplicationContext 的参数传入，而Spring容器在调用setBeanDactory前需要程序员自己指定（注入）setBeanDactory里的参数BeanFactory )
- 6、如果Bean实现了BeanPostProcess接口，Spring将调用它们的`postProcessBeforeInitialization（预初始化）方法`（作用是在Bean实例创建成功后对进行增强处理，如对Bean进行修改，增加某个功能）
- 7、如果Bean实现了InitializingBean接口，Spring将调用它们的`afterPropertiesSet`方法，作用与在配置文件中对Bean使用`init-method`声明初始化的作用一样，都是在Bean的`全部属性设置成功后执行`的初始化方法。
- 8、如果Bean实现了BeanPostProcess接口，Spring将调用它们的`postProcessAfterInitialization（后初始化）方法`（作用与第六步的一样，只不过是在Bean初始化前执行的，而这个是在`Bean初始化后`执行的，时机不同 )
- 9、经过以上的工作后，Bean将`一直驻留`在应用上下文中给应用使用，直到应用上下文被销毁
- 10、如果Bean实现了DispostbleBean接口，Spring将调用它的`destory`方法，作用与在配置文件中对Bean使用`destory-method`属性的作用一样，都是在Bean实例销毁前执行的方法。

综合上面十个步骤，可以简化为：`Bean实例的创建、Bean属性的注入、Bean相关Aware接口的实现，Bean实例调用前的初始化、Bean实例的调用、Bean实例销毁前的初始化、Bean实例的销毁`

&nbsp;

