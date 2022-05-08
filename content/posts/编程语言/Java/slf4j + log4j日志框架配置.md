+++
author = "pikachu"
title = "slf4j + log4j日志框架配置"
date = "2018-12-29"
description = " "
tags = [
    "",
	"log"
]
categories = [
    "it","java"
]

+++



## slf4j简介

> slf4j是一套包装Logging 框架的界面程式，以外观模式实现。可以在软件部署的时候决定要使用的 Logging 框架，目前主要支援的有**Java Logging API**、**log4j**及**logback**等框架。slf4j必须与其他日志库配合才能使用，一般需要将<b>抽象层（例如slf4j-api-xx.jar）+中间层（例如slf4j-log4j12）+实现层（例如log4j）</b>这三层都配置好才能保证SLF4J正常运行。

&nbsp;

![v2-d8261ab119309917e25180ef0ae42abe_r](https://user-images.githubusercontent.com/38284818/50534911-a72c6480-0b7e-11e9-8496-19853b139bae.jpg)

&nbsp;

## maven依赖

```
<!-- slf4j日志依赖包 -->

<!-- 抽象层依赖 -->
<dependency>
	<groupId>org.slf4j</groupId>
	<artifactId>slf4j-api</artifactId>
	<version>1.8.0-beta0</version>
</dependency>

<!-- 中间层和实现层依赖 -->
<dependency>
	<groupId>org.slf4j</groupId>
	<artifactId>slf4j-log4j12</artifactId>
	<version>1.8.0-beta0</version>
</dependency>
```

&nbsp;

## log4j.properties配置

- log4j支持两种文件类型的配置：**XML格式的文件**和**properties属性文件**，一般是将文件放置于src/main/resources/目录下。

- 配置中主要包括三个方面：日志的输出位置**Appenders**（控制台，文件，数据库等），日志的输出样式**Layouts**（HTML样式、自由指定样式等），日志根配置**Logger**（日志记录的最低等级等）；

- 详细配置信息参考：https://www.jianshu.com/p/ccafda45bcea

&nbsp;

## 问题及解决

- **异常**：java.lang.NoClassDefFoundError: org.slf4j.LoggerFactory
找了一个上午，以为是版本冲突或maven导入的jar包不完整，怎么改都不成功，绝望脸T_T，最后终于找到了解决方法，需要将maven依赖部署到程序集中。&nbsp;

- **解决方法**：
<b>右键项目—>“ Properties ”—>选择“ Deployment Assembly ”—>点击 “ add ” —>" Java Build Path Entries "—>" Maven Dependencies " </b>

- **其他问题1：**：如果创建servlet后或者改servlet的类名后出现servlet无法访问到的问题（404），将项目clean下后即可正常访问

- **其他问题2：**：如果mave-update后jre一直回退到1.5版本，可以在pom中加入下面配置进行设置，版本号对应自己的jdk版本
```
<build>
	<plugins>
		<plugin>
			<groupId>org.apache.maven.plugins</groupId>
			<artifactId>maven-compiler-plugin</artifactId>
			<version>3.1</version>
			<configuration>
				<source>1.8</source>
				<target>1.8</target>
			</configuration>
		</plugin>
	</plugins>
</build>
```

&nbsp;

参考网址：https://zhuanlan.zhihu.com/p/32475568