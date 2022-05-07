+++
author = "pikachu"
title = "Redis安装及使用"
date = "2019-01-20"
description = "Lorem Ipsum Dolor Si Amet"
tags = [
	"java",
	"中间件"
]
categories = [
    "IT"
]
+++


## 一、redis的安装（Linux环境）

1. 下载需要的redis版本：http://download.redis.io/releases/ ，将redis压缩包解压并进入其根目录`tar -zxvf redis-4.0.8.tar.gz -C /home/bigDataPresentationSys/
`，执行`make`命令进行编译。
    - 如果出现各种坑爹问题，参考这篇：https://blog.csdn.net/jy0902/article/details/19248299
    - **坑爹一**：注意允许的用户，我是在**root**用户下才运行成功的（应该跟**权限**有关）
    - **坑爹二**：尽量用官方**稳定版本**的，减少入一些奇奇怪怪的坑

&nbsp;

2. 执行`make test`命令检测是否编译成功，在执行命令的时候可能会出现`You need tcl 8.5 or newer in order to run the Redis test`的错误，通过`yum install tcl`进行tcl的安装并重新编译即可。

![1](https://user-images.githubusercontent.com/38284818/51435850-5ced5d80-1cbc-11e9-9c87-e945e5218407.JPG)

&nbsp;

## 二、redis的基本操作
1. **redis服务的启动及关闭**
    - 进入redis根目录下的src文件夹，然后运行`./redis-server`启动redis服务端，键盘按`control + C`关闭服务，或者输入命令`kill -9 PID`（PID为启动redis的PID），或者使用客户端的命令——`./redis-cli -h 127.0.0.1 shutdown`进行关闭
    &nbsp;
    ![2](https://user-images.githubusercontent.com/38284818/51435854-6e366a00-1cbc-11e9-9b58-b5b63e11b59b.JPG)
    
	&nbsp;
2. **redis的持久化及及基本的数据操作**
    - 通过redis-cli的shutdown的话redis会进行持久化（存储到磁盘上），即下次启动数据还在，若用`control + C`关闭redis，即存储的数据会消失。在client执行`save`命令的话，可以进行人工的持久化操作
    - redis的src目录下执行`./redis-cli`启动客户端（redis-server已启动的情况，执行命令`ping`，若返回`pong`则连接成功），通过`set key value`进行数据的设置，通过`get key`进行相应键值的获取，通过`keys *`获取所有的key
    
	&nbsp;
3. **redis修改启动的端口号**
    - 通过执行指定执行的**端口**进行修改：
        - `./redis-server --port 6380`：启动redis服务
        - `./redis-cli -p 6380`：启动redis客户端
        - `./redis-cli -p 6380 shutdown`：关闭redis服务
    - 通过执行指定的**配置文件**进行启动：
        - 修改根目录下的`redis-conf`文件，将port（端口）进行修改，执行命令
        `./redis-server ../redis-conf`，即可根据配置文件进行启动
    
	&nbsp;
4. **redis修改启动密码**
    - 在redis根目录下执行`sudo vim redis-conf`命令进行配置文件编辑，通过`/ + requirepass(搜索的单词)`进行搜索，`n`可以匹配下一个，`shift + n`可以匹配上一个，找到位置后修改密码，即redis客户端启动的时候需要输入密码才能启动，如`./redis-cli -a 1991111`
    ![image](https://user-images.githubusercontent.com/38284818/51436450-5369f280-1cc8-11e9-8715-e7e433f3364c.png)

