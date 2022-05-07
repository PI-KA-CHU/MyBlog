+++
author = "pikachu"
title = "单机部署Tomcat集群"
date = "2019-01-07"
description = "Lorem Ipsum Dolor Si Amet"
tags = [
    "java"
]
categories = [
    "IT",
]
+++


## 一、Tomcat的安装


- 单个Tomcat的安装（包括基本**JDK**，**Mysql**环境搭建）具体可参考这边笔记#20 ，现在需要将多个Tomcat安装到服务器上（下面安装的是两个），按照单个安装那样，通过Linux命令将Tomcat安装到**不同目录**下：

![2](https://user-images.githubusercontent.com/38284818/50751735-09275100-1286-11e9-8e5d-09adf3e2e030.JPG)


&nbsp;
&nbsp;


## 二、Tomcat的端口的修改


> 修改不同Tomcat的端口以避免同时启动时的冲突，**第一个Tomcat**可以不做修改，保持**默认端口**即可，多个Tomcat的端口可以按照+1000的形式进行添加，比如默认端口为8080，则第二个Tomcat的端口可设置为9080，方便记忆。


**1. 命令行打开的server.xml文件：（第二个Tomcat）**

`sudo vim /usr/java/tomcat2/apache-tomcat-8.5.35/conf/server.xml`

**2. 修改三个位置的端口号：（只修改port属性的端口即可）**

- 默认为**8005**，用于监听Tomcat的关闭

![1](https://user-images.githubusercontent.com/38284818/50753632-75598300-128d-11e9-8f95-0a3868b30322.JPG)

- 默认为**8080**，用于监听客户端请求，如果发送的是**https 请求**. 就将请求转发到**8443** 端口.，加入`URIEncoding=“UTF-8”`可以防止Tomcat**乱码**

![2](https://user-images.githubusercontent.com/38284818/50753636-77bbdd00-128d-11e9-86c0-5e026d115277.JPG)

- 默认为**8009**，用于接收其他服务器转发过来的请求

![3](https://user-images.githubusercontent.com/38284818/50753647-7d192780-128d-11e9-9d92-f746730a14ea.JPG)


&nbsp;
&nbsp;


## 三、系统环境变量的修改


- 为了防止多个Tomcat冲突问题，将安装的Tomcat路径都加入系统环境变量中，并通过修改Tomcat的server.xml配置其启动路径。

- CentOS目录下`/etc/profile`文件，在其最底部加入下面代码，下面代码的等号右边都是自己安装的**Tomcat的路径**：

（**CATALINA_BASE，CATALINA_HOME，TOMCAT_HOME**是Tomcat的默认启动路径，如果想要修改其启动路径，需要在Tomcat的**server.xml**文件中进行配置）


```
#set tomcat environment
export CATALINA_BASE=/usr/java/tomcat1/apache-tomcat-8.5.35
export CATALINA_HOME=/usr/java/tomcat1/apache-tomcat-8.5.35
export TOMCAT_HOME=/usr/java/tomcat1/apache-tomcat-8.5.35

export CATALINA_2_BASE=/usr/java/tomcat2/apache-tomcat-8.5.35
export CATALINA_2_HOME=/usr/java/tomcat2/apache-tomcat-8.5.35
export TOMCAT_2_HOME=/usr/java/tomcat2/apache-tomcat-8.5.35
```

- 修改完成并保存后需要更新配置，可以使用下面命令进行更新，或者重启服务器：

`source /etc/profile`


&nbsp;

## 四、Tomcat启动路径的修改

> 用linux的vim命令编辑**第二个Tomcat**（第一个Tomcat使用默认路径即可，可以不做修改）安装目录下的bin/**catalina.sh**文件，或者直接用模拟器内置的Text编辑器打开，找到如下位置（红框内的为需要加入的代码）：

![3](https://user-images.githubusercontent.com/38284818/50752640-e860fa80-1289-11e9-8d10-5d91e7dbce19.JPG)

&nbsp;

1. **文件打开命令：**

`sudo vim /usr/java/tomcat2/apache-tomcat-8.5.35/bin/catalina.sh`


2. **加入代码**（等号右边为上一步配置的相应Tomcat的**环境变量**，修改后保存）：

```
CATALINA_BASE=$CATALINA_2_BASE
CATALINA_HOME=$CATALINA_2_HOME
```

&nbsp;


## 五、巨坑回顾及学习成果


**1. 填坑总结：**

- 如果要修改Tomcat端口，需要**先关闭tomcat，再修改端口**，否则可能出现端口一直被占用的情况，此时只能重启服务器（个人方法）。

- 进行集群tomcat配置的时候，需要修改**防火墙入站规则**，即加入允许访问的端口（阿里云服务器可以在**控制台**的入站规则里修改），否则一直出现503！！！

**2. 学习成果：**

- `netstat -an | grep 8080`：可以查看端口的占用情况

- `sudo vim server.xml`：以默认权限（管理员）修改文件

- `echo $CATALINA_BASE`：可以查看环境变量是否生效

- `reboot`：重启服务器

- `source /etc/profile`：更新配置文件，使得配置生效



