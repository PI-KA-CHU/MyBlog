+++
author = "pikachu"
title = "Maven + SpringBoot实现环境隔离"
date = "2019-01-02"
description = "Lorem Ipsum Dolor Si Amet"
tags = [
    "java",
	"spring-boot"
]
categories = [
    "IT",
]
+++


## 一、Maven环境隔离简介

#### 基本类别
- 本地开发环境（local）
- 开发环境（dev）
- 测试环境（beta）
- 线上环境（prod）

#### 解决的问题

- 避免人工修改配置信息导致的问题
- 轻松分环境编译，打包，部署
- ......

#### 环境隔离的依赖

- **Spring Profile**：
	- a. Spring可使用Profile决定程序在不同环境下执行情况，包含配置、加载Bean、依赖等。
	Spring的Profile一般项目包含：**dev(开发), test(单元测试), qa(集成测试), prod(生产环境)**。由**spring.profiles.active**属性决定启用的profile。
	
	- b. SpringBoot的配置文件默认为 **application.properties**(或yaml，此外仅以properties配置为说明)。不同Profile下的配置文件由**application-{profile}.properties**管理，同时独立的 Profile配置文件会覆盖默认文件下的属性。

- **Maven Profile：**
Maven同样也有Profile设置，可在构建过程中针对不同的Profile环境执行不同的操作，包含配置、依赖、行为等。
Maven的Profile由 **pom.xml** 的标签管理。每个Profile中可设置：id(唯一标识), properties(配置属性), activation(自动触发的逻辑条件), dependencies(依赖)等。

&nbsp;

## 二、相关配置

- **pom.xml**中，在build（dependencies）同级的地方加入：
```
//分别代表开发（dev），测试（beta）和线上（prod）环境
<profiles>
    <profile>
        <id>dev</id>
        <activation>
            <activeByDefault>true</activeByDefault>
        </activation>
        <properties>
            <profile>dev</profile>
        </properties>
    </profile>
    <profile>
        <id>beta</id>
        <properties>
            <profile>beta</profile>
        </properties>
    </profile>
    <profile>
        <id>prod</id>
        <properties>
            <profile>prod</profile>
        </properties>
    </profile>
</profiles>
```

- 在classpath下分别创建三个环境需要的**application-${profile}.propertices：**

![default](https://user-images.githubusercontent.com/38284818/50625172-3790ed80-0f61-11e9-9502-ce07352c2e69.JPG)


- 在**application.propertices**中配置指定的环境：

```
//配置动态的profile注入（profile的值由maven修改）
spring.profiles.active=@profile@
```

&nbsp;
<h3>3. 通过Eclipse对maven的profile进行设置：</h3>
（右键项目 - peoperties - maven - 输入pom中配置的profile的Id - Apply后即配置环境成功）

![2](https://user-images.githubusercontent.com/38284818/50625473-67d98b80-0f63-11e9-9e2b-10005520d02e.JPG)

&nbsp;

#### 参考文章：
- https://www.jianshu.com/p/948c303b2253
- https://yq.aliyun.com/articles/670083
- http://tengj.top/2017/02/28/springboot2/
- https://www.jianshu.com/p/ad1b8157feaf

<hr>
