+++
author = "pikachu"
title = "SpringBoot创建及配置（一）"
date = "2018-12-02"
description = "Lorem Ipsum Dolor Si Amet"
tags = [
    "java",
	"spring-boot"
]
categories = [
    "IT",
]
+++


## 创建流程

1. 创建Maven项目，设置Maven架构类型为**maven-archetype-quickstart**：

![default](https://user-images.githubusercontent.com/38284818/49331420-6c62fb00-f5d7-11e8-914a-3408001a0c65.JPG)

&nbsp;

2. 创建项目后，修改其**pom.xml**文件，如下：

```
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.zeng</groupId>
	<artifactId>myspring-boot</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>war</packaging>

	<name>myspring-boot</name>
	<url>http://maven.apache.org</url>

	<!-- spring Boot的基本组件依赖 -->
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.0.0.RELEASE</version>
	</parent>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
	</properties>
	

	<dependencies>
		<!-- 单元测试依赖 -->
                <dependency>
		   <groupId>org.springframework.boot</groupId>
		   <artifactId>spring-boot-starter-test</artifactId>
		   <scope>test</scope>
		</dependency>
		<dependency>
		   <groupId>com.jayway.jsonpath</groupId>
		   <artifactId>json-path</artifactId>
		</dependency>
		
		<!-- 数据库依赖包 -->
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
		</dependency>
		<!-- Jpa依赖包，用于对数据库的操作 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-jpa</artifactId>
		</dependency>
		
		<!--WEB支持-->
		<dependency>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-web</artifactId>
		    <exclusions>
		        <exclusion>
		            <groupId>org.springframework.boot</groupId>
		            <artifactId>spring-boot-starter-tomcat</artifactId>
		        </exclusion>
		    </exclusions>
               </dependency>
        
		<!--用于编译jsp-->
		<dependency>
		    <groupId>org.apache.tomcat.embed</groupId>
		    <artifactId>tomcat-embed-jasper</artifactId>
		    <scope>provided</scope>
		</dependency>
        
              <!-- Spring-boot实现热部署支持包 -->
               <dependency>
	             <groupId>org.springframework.boot</groupId>
	             <artifactId>spring-boot-devtools</artifactId>
	             <optional>true</optional>
    	       </dependency>
        
              <dependency>
		   <groupId>io.github.biezhi</groupId>
		   <artifactId>wechat-api</artifactId>
		   <version>1.0.6</version>
	     </dependency>
		
	<!-- Mybatis支持 -->
	<dependency>
	        <groupId>org.mybatis.spring.boot</groupId>
	        <artifactId>mybatis-spring-boot-starter</artifactId>
	        <version>1.1.1</version>
    	</dependency>
    	
    	<!-- 打包成war包时需要移除内置tomcat -->
    	<dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-tomcat</artifactId>
               <scope>provided</scope>
        </dependency>
        
        <!-- servlet的api依赖 -->
	<dependency>
	        <groupId>javax.servlet</groupId>
		<artifactId>javax.servlet-api</artifactId>
	</dependency>
		
	<!-- 文件上传依赖包 -->
	<dependency>
		<groupId>commons-fileupload</groupId>
		<artifactId>commons-fileupload</artifactId>
		<version>1.3.1</version>
	</dependency>
	<dependency>
		<groupId>commons-io</groupId>
		<artifactId>commons-io</artifactId>
		<version>2.4</version>
	</dependency>

	</dependencies>
	
	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
			<!-- 在打包war时引入第三包jar包 -->
			<plugin>
                                <groupId>org.apache.maven.plugins</groupId>
                                <artifactId>maven-war-plugin</artifactId>
                                <configuration>
                                      <webResources>
                                      <resource>
                                             <directory>src/main/webapp</directory>
                                             <targetPath>WEB-INF/lib/</targetPath>
                                             <includes>
                                                     <include>**/*.jar</include>
                                             </includes>
                                   </resource>
                                   </webResources>
                               </configuration>
                     </plugin> 
		</plugins>
	</build>
	
</project>

```
&nbsp;

3. 在項目的/resources下配置**application.properties**（主要包括数据库jdbc，热部署，MyBatis映射文件地夹设置等），如下：

```
######  data config start ###### 
spring.jpa.properties.hibernate.hbm2ddl.auto=update
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect
spring.jpa.show-sql= true
spring.datasource.hikari.connectionInitSql=SET NAMES utf8mb4 COLLATE utf8mb4_unicode_ci;

######   DB config end ###### 
spring.datasource.url=jdbc:mysql://127.0.0.1:3306/lost_back?useSSL=false
spring.datasource.username=root
spring.datasource.password=19970121
spring.datasource.driver-class-name=com.mysql.jdbc.Driver


######   Mapper config
mybatis.mapper-locations=classpath:mapper/*.xml

# Page default prefix directory
server.servlet.jsp.init-parameters.development=true
spring.mvc.view.prefix=/WEB-INF/jsp/
# Response page default suffix
spring.mvc.view.suffix=.jsp


# Configuring hot deployment
spring.devtools.restart.enabled: true
# Set the restart directory   do not restart the folder WEB-INF/**
spring.devtools.restart.exclude: WEB-INF/**
spring.thymeleaf.cache=false
```