 +++
author = "pikachu"
title = "Tomcat配置日志格式"
date = "2019-03-30"
description = "Lorem Ipsum Dolor Si Amet"
draft = false
tags = [
    "log"
]
categories = [
    "IT"
]
+++



## 前言

查看日志是重要的排查异常的方式，在出现error时查找日志可以快速确定出错的原因，而在大数据时代里，随着用户的访问上升，日志数据量也呈爆炸式上升，通过日志可以分析出用户的行为特征，如兴趣，情感，访问时间等，可以通过接口响应时间分析用户体验等，从而为系统制定调优方案。日志在大数据时代下的发挥着重要的作用，日志的规范及格式则是获取日志数据的关键，通过设置写入的日志文件使得在进行数据挖掘时能够获取到相应的数据。

<br>

## 日志配置 - `localhost_access_log`

<br>

- 进入tomcat安装目录下的`/conf/server.xml`文件，找到如下文段并将其取消注释
```
<Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
               prefix="localhost_access_log" suffix=".txt"
               pattern="%h %l %u %t &quot;%r&quot; %s [agent- %{User-Agent}i -] %b %Dms" fileDateFormat="yyyy-MM-dd.HH.mm" />
```


**属性**：

![image](https://user-images.githubusercontent.com/38284818/55274923-2ca7e800-5319-11e9-9366-3c76d446ebf6.png)


**pattern属性的参数**：

![image](https://user-images.githubusercontent.com/38284818/55274950-8c9e8e80-5319-11e9-94be-8a5671b5aaa9.png)

<br>

- **设置日志的生成周期**

    通过设置`fileDateFormat="yyyy-MM-dd.HH.mm"`属性的值可以设置日志的生成时间，如`yyyy-MM-dd.HH.mm`表示每分钟生成一次，`yyyy-MM-dd`表示每天生成一次
    
<br>

- **访问者IP设置**
    - 正常情况下：`pattern="%h"`
    - nginx情况下：`pattern="%{X-Real-IP}i"`(根据nginx中的配置修改)

<br>

- **设置User-Agent**
    - `pattern="%{User-Agent}i"`
    

<br>

## 参考
- https://blog.csdn.net/qq_30121245/article/details/52861935
- https://jingyan.baidu.com/article/36d6ed1f713b9d1bcf4883e1.html
- https://yq.aliyun.com/articles/518410