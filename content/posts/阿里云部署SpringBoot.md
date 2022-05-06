+++
author = "pikachu"
title = "阿里云部署SpringBoot"
date = "2018-11-29"
description = "Lorem Ipsum Dolor Si Amet"
tags = [
    "开发工具",
    "spring-boot"
]
categories = [
    "IT",
]
+++


## 一、部署流程

1. **阿里云服务器购买**（学生优惠）：https://promotion.aliyun.com/ntms/act/campus2018.html
购买成功后可以在阿里云服务器控制台上管理自己的服务器，设置自己的登陆账号和密码。<br><br>

2. 使用**终端模拟器MobaXterm访问远程服务器**，点击左上角的session，后点击SSH进行连接，连接地址为你的公网网址。<br>
![default](https://user-images.githubusercontent.com/38284818/49203549-ad92b980-f3e3-11e8-8a1c-3beba26f0b21.JPG)<br><br>


3. 连接成功后在MoBaXterm上传jdk及tomcat，并配置环境变量，参考如下：

- http://www.cnblogs.com/yijialong/p/9606265.html

- https://www.cnblogs.com/yuanbo123/p/5819564.html<br><br>

4. 配置成功后，需要在阿里云控制台那里设置**防火墙（入站规则）** ，比如开放**3306**端口供**Mysql**数据库，开放**8080**供**Tomcat**使用（**tomcat默认是8080端口**），不开放tomcat端口的话访问到的是503错误

![default](https://user-images.githubusercontent.com/38284818/50751361-691cf800-1284-11e9-99fc-63a209f9b601.JPG)

&nbsp;

## 二、问题与解决

> 使用SpringBoot的jar包进行部署时发生了一堆无法描述的事情，最终改用war包进行打包部署，以下是遇到的一些坑：


1. 坑爹的tomcat无法启动：jdk异常：

- https://blog.csdn.net/kaikaitang/article/details/80888664


2. 在项目根目录下进行maven的war打包（使用maven的war打包时需要使用Eclipse的JDK若使用的是JRE则需要修改过来）：

- **mvn clean package -Dmaven.test.skip=true**（后面部分为忽略test文件）

3. 配置完jdk和Tomcat并成功打包war包后，还需要**配置数据库**，我的是MySQL，否则即使war包成功部署了，也**无法正常访问项目（404）**，具体操作可根据此：

	- Mysql配置及安装： http://www.cnblogs.com/yijialong/p/9606265.html

	- Mysql编码格式修改： https://blog.csdn.net/qq_28039297/article/details/76686022

4. 一切准备就绪后，可以使用**MobaXterm进行远程终端连接**，通过该工具将war包上传到Tomcat的webapps目录下，运行tomcat就可以在公网进行访问了。（若无法正常访问，可查看**tomcat日志文件**寻找异常错误，在其logs文件夹中）

5. 使用**navicat将本地数据库导成.sql文件后导入服务器端数据库**出现的坑爹问题，navicat在导出时默认不是**65001 (UTF-8)**，导入的时候要用**Current Windows Codepage**编码格式进行导入，否则会出现navicat显示正常，但是使用web访问时会出现乱码的问题


![sql](https://user-images.githubusercontent.com/38284818/50414727-17bb4480-0852-11e9-9185-29c7a34a9bcd.JPG)


