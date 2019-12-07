+++
author = "pikachu"
title = "Docker实战：3、三剑客之Docker Compose"
date = "2019-08-22"
description = "Lorem Ipsum Dolor Si Amet"
tags = [
	"docker",
	"开发工具"
]
categories = [
    "IT"
]
+++


## 一、前言
> **Docker Compose和Dockerfile的区别**：
Dockerfile主要是记录了一个镜像的构建过程，可以达到快速构建项目的目的，而Docker Compose则是构建项目（服务）的过程，一个服务中可以有多个容器。docker-compose.yml不提供镜像的构建信息，而是通过Dockerfile进行获取（需要build的情况）。

&nbsp;

## 二、Docker Compose的安装


```
# 下载安装
curl -L https://get.daocloud.io/docker/compose/releases/download/1.20.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose

# 添加权限
chmod +x /usr/local/bin/docker-compose

#查看版本
docker-compose version
```

&nbsp;

## 三、搭建项目环境


**3.1、项目目录结构**

- docker目录下每个文件夹对应一个服务，并且有各自的`Dockerfile`（用于该服务的镜像构建）及相关挂载的数据，`docker-compose.yml`则控制各个服务的镜像构建，数据挂载，命令启动及启动流程等，通过`docker-compose`命令实现快速部署启动。

![image](https://user-images.githubusercontent.com/38284818/63484649-f3903000-c4d2-11e9-8118-65588e3ab32a.png)


**3.2、镜像文件详情**

**a、Dockerfile:**
```
app:
FROM maven:3.5-jdk-8

mysql：
FROM mysql/mysql-server:5.7

redis：
FROM redis:5.0.5

rocketmq：未整合

```

**b、docker-compose.yml：**
```
version: '3'
services:
  nginx:
    container_name: bigdata-nginx
    image: nginx:1.16.1
    restart: always
    ports:
      - 80:80
      - 443:443
    volumes:
      - ./nginx/conf.d:/etc/nginx/conf.d
      - /tmp/logs/nginx:/var/log/nginx

  mysql:
    container_name: bigdata-mysql
    build: mysql
    # command: --default-authentication-plugin=mysql_native_password
    environment:
      MYSQL_DATABASE: multisource_bigdata
      MYSQL_ROOT_PASSWORD: root
      MYSQL_ROOT_HOST: '%'
      MYSQL_USER: 'hive'
      MYSQL_PASS: 'hive'
      TZ: Asia/Shanghai
    ports:
      - 3306:3306
    volumes:
      - ./mysql/mysql_data:/var/lib/mysql
    restart: always

  redis:
    container_name: bigdata-redis
    build: redis
    working_dir: /redis
    volumes:
      - ./redis/redis.conf:/etc/redis/redis.conf
    restart: always
    ports:
      - 6379:6379
    command: redis-server /etc/redis/redis.conf

  app:
    container_name: bigdata-app
    build: app
    volumes:
      - ././:/app
      - ~/.m2:/root/.m2
      - /tmp/logs/app:/usr/local/logs
    working_dir: /app
    restart: always
    expose:
      - 8080
    command: mvn clean spring-boot:run -Pprod -Dmaven.test.skip=true
    depends_on:
      - nginx
      - mysql
      - redis
```
**c、/nginx/conf.d/app.conf**：nginx的挂载文件
```
server {
    listen 80;
    charset utf-8;
    access_log off;

    location / {
        proxy_pass http://app:8080;
        proxy_set_header Host $host:$server_port;
        proxy_set_header X-Forwarded-Host $server_name;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

    location /static {
        access_log   off;
        expires      30d;

        alias /app/static;
    }
}

```
**d、/mysql/mysql_data**：mysql数据库数据的挂载

**e、/redis/redis.conf**：redis配置文件的挂载

- 比如我们先在本地配置redis，并成功整合项目后，可以将原来的配置文件挂载到容器的redis配置文件下，从而快速实现整合

&nbsp;

## 四、快速部署并启动项目


```
1、cd 进入项目 /src/docker 目录
2、执行 `docker-compose up` (会出现相关的启动图标。如redis、springboot等)
```

![image](https://user-images.githubusercontent.com/38284818/63485632-b4fc7480-c4d6-11e9-8d18-696d767f4f6f.png)

```

3、访问项目的index界面，测试nginx：
    - http://192.168.125.131/index.html
    - （resource/static/index.html）
4、访问接口，测试数据库连接及redis缓存（接口实现缓存的情况）
    - http://192.168.125.131/api/test/getUser

```

![image](https://user-images.githubusercontent.com/38284818/63485717-191f3880-c4d7-11e9-91ec-ab4ad1aa54d2.png)


```
5、容器内部访问（非必须）

# 查看容器Id
docker ps -a
# 进入容器内部
docker exec -it 容器Id bash

```

&nbsp;


## 五、问题与解决：


**问题一**：执行`docker-compose up`后异常
```
container_linux.go:345: starting container process caused "exec: \"mvn\": executable file not found in $PATH": unknown
```
**解决**：

- 参考：https://medium.com/p/baf30968163e/responses/show
```
构建的docker容器中没有maven环境，无法通过maven进行打包，将web项目的镜像改为 maven:3.5-jdk-8 ，此镜像包含了jdk1.8 + maven3.5，即可正常运行
```

&nbsp;

**问题二**：修改mysql密码不生效问题

**解决**：
```
重启mysql的docker服务
```

**问题三**：navicat无法正常连接docker数据库

**解决**：

- 本人问题：mysql的映射端口弄错了，搞了半天，坑爹啊
- 其他参考：https://blog.csdn.net/liqz666/article/details/82225575

**问题四**：无法正常连接服务，如redis、mysql等

**解决**：

- 参考：https://blog.csdn.net/harris135/article/details/74167910

- 方案一：将相关端口添加到防火墙开放：
```
# 添加开放端口
firewall-cmd --zone=public --add-port=80/tcp --permanent  

# 重启防火墙（不重启可能无法生效）
systemctl restart firewalld.service
```
- 方案二：关闭防火墙：`systemctl stop firewalld.service`

&nbsp;
&nbsp;

## 六、参考

- http://www.ityouknow.com/docker/2018/03/22/docker-compose.html
- https://www.jianshu.com/p/efd70ad53602
- http://www.ityouknow.com/springboot/2018/04/02/docker-favorites.html
- http://www.ityouknow.com/springboot/2018/03/28/dockercompose-springboot-mysql-nginx.html
