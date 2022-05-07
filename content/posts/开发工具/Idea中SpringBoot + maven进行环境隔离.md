+++
author = "pikachu"
title = "Idea中SpringBoot + maven进行环境隔离"
date = "2019-02-08"
description = "Lorem Ipsum Dolor Si Amet"
tags = [
	"java",
	"spring-boot",
	"开发工具"
]
categories = [
    "IT"
]
+++


## 1. 新建各个环境的配置文件夹
![image](https://user-images.githubusercontent.com/38284818/52459314-61ac7f80-2b9f-11e9-92b6-ec2d5cd1b1ca.png)

&nbsp;

## 2. pom.xml中配置profile
```
<!-- Maven多环境隔离配置 -->
<project>
    <build>
        <!-- 多环境文件夹配置，根据profile选择对应的文件夹 -->
    	<resources>
    		<resource>
    			<directory>src/main/resources</directory>
    			<!-- 排除指定文件目录下的配置文件 -->
    			<excludes>
    				<exclude>config_dev/*</exclude>
    				<exclude>config_prod/*</exclude>
    				<exclude>config_beta/*</exclude>
    			</excludes>
    			<filtering>true</filtering>
    		</resource>
    		<!-- 通过profile指定配置的文件夹 -->
    		<resource>
    			<directory>src/main/resources/config_${profile}</directory>
    		</resource>
    	</resources>
    </build>
    
    <!-- 配置不同的环境 -->
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
</project>
```

&nbsp;

## 3. 配置application.properties
- SpringBoot会根据选择的`profile`选择对应的配置文件：`application-${profile}.properties`
```
# Multiple environment configuration(beta,prod,dev)
spring.profiles.active=@profile@
```

&nbsp;

## 4. 配置不同环境对应文件夹中的application-${profile}.properties文件
- 通过springBoot的`spring.profiles.include=dataBase`可以导入指定的配置文件（多个用逗号隔开），指定的配置文件名需要以`application-${profile}.properties`进行命名
![image](https://user-images.githubusercontent.com/38284818/52459902-c0bfc380-2ba2-11e9-8d62-cb6afd9fda77.png)

&nbsp;

## 5. 通过Idea选择profile进而自动配置对应的环境配置
- 此处分为`dev-开发环境`，`prod-线上环境`，`beta-测试环境`
![image](https://user-images.githubusercontent.com/38284818/52459073-0b8b0c80-2b9e-11e9-9d0f-5c343b84b84f.png)
