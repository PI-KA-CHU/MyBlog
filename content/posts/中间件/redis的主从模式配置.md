+++
author = "pikachu"
title = "redis的主从模式配置"
date = "2019-02-25"
description = " "
tags = [
	"redis"
]
categories = [
    "it","中间件"
]

+++


## 前言

- `replica-master`模式就是`slave-master`模式，查了一下好像是slave-master名称遭到太多人反对，作者被迫进行了修改。

- 配置了主从模式后重启相应配置的redis服务器，`从redis会异步复制主redis的数据`，开启`AOF`可以实现数据的持久性（将数据写入磁盘）

- 下面主从模式的配置是建立在已经启动redis-cluster的前提上进行修改配置，集群搭建详情参考#52 。

- **主从、哨兵和集群的关系**：
    - `主从`：**数据备份**及**读写分离**
    - `哨兵`：**高可用**，主挂了哨兵可以**选主**切换
    - `集群`：**数据 hash 分片**， 解决单台机器**资源的上限的问题**，**分散压力**。


&nbsp;

## redis的复制
- Redis使用`异步复制`，异步从站到主站确认处理的数据量。

- 主设备可以有`多个从设备`。

- 复制在从属端也很大程度上是`非阻塞的`。当slave正在执行初始同步时，假设您在redis.conf中配置了Redis，它可以使用旧版本的数据集处理查询。

- 复制可用于`可伸缩性`，以便为只读查询提供多个从站（例如，可以将慢速O（N）操作卸载到从站），或者仅用于提高数据安全性和高可用性。

- 可以使用复制来避免让主服务器将完整数据集写入磁盘的成本：典型的技术是配置主服务器redis.conf以避免持久存储到磁盘，然后连接配置为不时保存的从服务器，或者已启用`AOF`。但是，必须小心处理此设置，因为重新启动的主服务器将以空数据集开始：如果从服务器尝试与其同步，则从服务器也将被清空。

&nbsp;

## 修改replica（从者redis）的`redis.conf`配置文件（有密码还需要配置replica连接master的密码）
```
# 设置master的IP及端口
replicaof [masterip] [port]

# replica需要关闭集群的启动模式，否则会报异常
cluster-enable no
```
&nbsp;

## 重启replica的redis
- 进入redis的src目录下，执行`./redis-cli -a [password] -h [ip] -p [port] shutdown`关闭redis

- 使用`./redis-server ../redis.conf`命令重启redis服务器

![image](https://user-images.githubusercontent.com/38284818/53335008-4e3a3c00-3935-11e9-9242-a54805b0a827.png)

&nbsp;

#### 参考文献

- 官方文档： https://redis.io/topics/replication

&nbsp;

