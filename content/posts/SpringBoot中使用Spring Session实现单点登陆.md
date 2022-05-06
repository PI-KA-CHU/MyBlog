+++
author = "pikachu"
title = "SpringBoot中使用Spring Session实现单点登陆"
date = "2019-02-13"
description = "Lorem Ipsum Dolor Si Amet"
tags = [
	"java",
	"spring-boot"
]
categories = [
    "IT"
]
+++


&nbsp;
## 一、Session的共享问题

> Session的共享可以通过`redis`的操作及`spring session`进行解决，从而实现单点登陆，redis操作可以直接通过调用`template`的方法进行数据的存取，但是其对业务具有一定的侵入性，使用`spring session`对session的存储进行封装，可以通过正常的HttpSession操作进行代码编写，其支持使用Redis、Mongo、JDBC、Hazelcast来存储Session，可以通过指定不同的存储方式进行存储，此处使用的是redis

&nbsp;

## 二、Spring-Session实现

#### Maven依赖
```
<!-- 引入redis依赖 -->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>

<!--spring session的redis相关依赖 -->
<dependency>
	<groupId>org.springframework.session</groupId>
	<artifactId>spring-session-data-redis</artifactId>
	<version>2.1.3.RELEASE</version>
</dependency>
```

#### 配置application.properties
```
spring.session.store-type=redis
```

#### 在启动类中加入@EnableRedisHttpSession 注解
```
@EnableRedisHttpSession
@SpringBootApplication
public class LostBackSysApplication extends SpringBootServletInitializer{

	
	@Override
	protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
		
		return application.sources(LostBackSysApplication.class);
	}
	
	public static void main(String[] args) {
//		System.setProperty("spring.devtools.restart.enabled", "false");
		SpringApplication.run(LostBackSysApplication.class, args);
	}

}

```


#### Spring Session的操作(与HttpSession操作无区别)
```
@RequestMapping(value = "/check",method=RequestMethod.POST)
public BaseReturnDto checkUser(HttpSession session) {
	
	//使用Spring Session存储至redis中
	session.setAttribute(‘key’,‘value’);
}
```

&nbsp;

## 三、使用过程中遇到的坑

- **异常**：
```
Error starting ApplicationContext. To display the conditions report re-run your application with 'debug' enabled.
2018-05-11 22:27:32.921 ERROR 5600 --- [ main] o.s.boot.SpringApplication : Application run failed

org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'enableRedisKeyspaceNotificationsInitializer' defined in class path resource [org/springframework/session/data/redis/config/annotation/web/http/RedisHttpSessionConfiguration.class]: Invocation of init method failed; nested exception is java.lang.NoSuchMethodError: org.springframework.data.redis.connection.RedisConnection.getConfig(Ljava/lang/String;)Ljava/util/List;
at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.initializeBean(AbstractAutowireCapableBeanFactory.java:1706) ~[spring-beans-5.0.6.RELEASE.jar:5.0.6.RELEASE]
at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(Abstr
...

```

- **解决**：
    - **一**：spring session的redis依赖版本不对
    - **二**：依赖导入只需要springBoot对redis的依赖及spring session对redis的依赖两个即可，不要再导入其他spring session依赖。

&nbsp;

#### 参考
- https://juejin.im/post/5bdd449b6fb9a04a09557a40
- https://www.jianshu.com/p/e4191997da56

&nbsp;
