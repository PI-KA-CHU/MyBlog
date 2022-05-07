+++
author = "pikachu"
title = "Nginx的安装及负载均衡（Linux环境）"
date = "2019-01-11"
description = "Lorem Ipsum Dolor Si Amet"
tags = [
    "java",
	"中间件"
]
categories = [
    "IT",
]
+++


## 一、依赖环境安装

**操作系统：** CentOS 7.3
**安装版本：** nginx-1.14.2

**安装gcc：**

`yum install gcc-c++`

**安装其他依赖：**

```
yum install -y pcre pcre-devel
yum install -y zlib zlib-devel
yum install -y openssl openssl-devel
```

&nbsp;

## 二、Nginx安装


#### 官网下载地址

- http://nginx.org/en/download.html

![2659012-c6605f8c454a66a3](https://user-images.githubusercontent.com/38284818/51006138-d67aa280-157c-11e9-9200-4c242e11866f.png)

&nbsp;

#### nginx下载包的解压

- Windows下载完后通过**MobaXtem**终端模拟器将安装包上传到服务器（CentOS），通过以下命令进行解压：

`tar -xzvf 压缩包文件地址 -C 解压缩到的文件夹`

&nbsp;

#### nginx的编译安装

- 进入nginx目录，执行如下命令：

```
# --prefix可以指定安装目录
./configure --prefix=/usr/java/nginx/nginx-1.14.2/
make & make install
```

&nbsp;

#### nginx的启动

- 编译成功后，nginx根目录下会出现**sbin文件夹**，进入sbin文件夹并**运行nginx**：`./nginx`

- 查看启动是否成功

```
# 启动完毕查看是否启动成功
ps -ef | grep nginx
```

&nbsp;

## 三、Nginx负载均衡的配置


<h3>进入nginx目录下的conf文件夹：</h3>
&nbsp;

**1.** 创建**vhost文件夹**，用于添加负载均衡的相关配置：

![2](https://user-images.githubusercontent.com/38284818/51007598-0d53b700-1583-11e9-9815-e520b4d404f5.JPG)

**2.** 进入vhost文件夹，创建<b>*.conf文件 </b>（如myconf.conf），加入如下配置：

```
#将到120.78.151.65的请求负载到不同端口的tomcat上，weight为其权重
upstream 120.78.151.65{
  server 120.78.151.65:18080 weight=10;
  server 120.78.151.65:9080 weight=1;
}

server{
  listen 80;  #监听的端口号
  autoindex on;
  server_name 120.78.151.65  #监听的服务器
  access_log /usr/java/nginx/nginx-1.14.2/logs/access.log combined;
  index index.jsp index.html index.htm index.php;

  #将端口监听到的请求转发到如下服务器，通过上面进行负载均衡
  location / {
      proxy_pass http://120.78.151.65;
  }
  
}

```

**3.** 在nginx的conf文件夹下的nginx.conf中加入如下代码`include vhost/*.conf`，将**vhost的配置文件**导入：

![default](https://user-images.githubusercontent.com/38284818/51007928-7f78cb80-1584-11e9-8fac-dab819236657.JPG)


**4.** 进入**sbin文件夹**

- **关闭nginx：**
`./nginx -s stop`

- **重启nginx：**
`./nginx -s reload`

- **查看启动是否成功：**
`./nginx -t`

![3](https://user-images.githubusercontent.com/38284818/51008235-ee0a5900-1585-11e9-854d-e1e465e7fa6c.JPG)


&nbsp;

## 四、Nginx安装的异常及解决：</h2>

#### 异常

```
nginx: [alert] could not open error log file: open() "/usr/local/nginx/logs/error.log" failed (2: No such file or directory)
2019/01/10 19:08:56 [emerg] 6996#0: open() "/usr/local/nginx/logs/access.log" failed (2: No such file or directory)
```


#### 原因

- nginx/目录下没有**logs文件夹**


#### 解决

1. 在**nginx根目录**下运行如下命令：

```
mkdir logs
chmod 700 logs
```

2. 创建完logs文件夹后**重新启动**nginx

&nbsp;

## 五、Nginx安装及负载均衡配置（Windows环境下）

#### 安装与配置
- 基本步骤与Linux环境相同，如果需要修改本地域名重定向用于测试负载均衡，打开`C:\Windows\System32\drivers\etc`目录下的hosts文件，加入`127.0.0.1 www.test.com`即可将www.test.com重定向到本地

#### 遇到的坑
- 安装nginx后需要**重启电脑**后配置的文件才能生效，否则nginx不会进行跳转

&nbsp;


#### 参考文献

- https://www.cnblogs.com/xxoome/p/5866475.html

- https://www.jianshu.com/p/deeb4bb5cedb

- https://www.jianshu.com/p/320a48fcef57


<hr>