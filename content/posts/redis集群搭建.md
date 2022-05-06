+++
author = "pikachu"
title = "redis集群搭建"
date = "2019-02-26"
description = "Lorem Ipsum Dolor Si Amet"
tags = [
	"java",
	"中间件"
]
categories = [
    "IT"
]
+++



## 一、集群与分布式的区别

> 小饭店原来只有一个厨师，切菜洗菜备料炒菜全干。后来客人多了，厨房一个厨师忙不过来，又请了个厨师，两个厨师都能炒一样的菜，这两个厨师的关系是集群。为了让厨师专心炒菜，把菜做到极致，又请了个配菜师负责切菜，备菜，备料，厨师和配菜师的关系是**分布式**，一个配菜师也忙不过来了，又请了个配菜师，两个配菜师关系是**集群**

- https://blog.csdn.net/bluishglc/article/details/5483162

&nbsp;

## 二、redis集群搭建过程（Linux环境）
- redis官方文档：（建议跟着文档走） https://redis.io/topics/cluster-tutorial

- 大佬博客： https://www.zybuluo.com/phper/note/195558

- redis集群搭建前提，修改`redis.conf`配置文件，找到以下配置信息并做如下图修改：
![image](https://user-images.githubusercontent.com/38284818/52361189-a12c7c00-2a78-11e9-9048-4586d4a5783b.png)

- redis集群至少需要开启**六个redis节点**，复制六个redis文件夹，在配置文件`redis.conf`中修改不同的端口号，分别进入src目录，执行命令`./redis.server ../redis.conf`，带配置文件启动redis。

![image](https://user-images.githubusercontent.com/38284818/52359928-d4b9d700-2a75-11e9-9579-2dbea872bab0.png)

- 在阿里云服务器（个人使用的服务器）中开启端口号，如`6379/6384`（6379-6384），开启后还需要开启集群总线端口`16379/16384` （16379-16384）,若未开启，在使用公网Ip进行集群搭建的时候会报错。
![image](https://user-images.githubusercontent.com/38284818/52360870-f3b96880-2a77-11e9-94ce-00c28f96b689.png)

- 进入其中一个redis的src文件夹中，执行命令`./redis-cli -a password --cluster create xxx.xx.xxx.xx:6379 xxx.xx.xxx.xx:6380 xxx.xx.xxx.xx:6381 xxx.xx.xxx.xx:6382 xxx.xx.xxx.xx:6383 xxx.xx.xxx.xx:6384 --cluster-replicas 1`创建集群，出现提示信息，输入`yes`后没有报异常则创建成功（如果已经创建过集群，需要先将每个redis的src目录下的`nodes.conf`删除，否则会有异常）。

- 创建成功后，可在redis的src目录下执行命令`./redis-cli -a password --cluster fix xxx.xx.xxx.xx:6379`进行检查。(下图表示创建成功)
![image](https://user-images.githubusercontent.com/38284818/52361803-0df44600-2a7a-11e9-8ad9-fb75bda8c15a.png)

- **如果是重新创建集群**，即使删除`nodes.conf`文件后可能创建的集群所有的slot没有被覆盖，需要在其中一个redis的src目录下执行命令`./redis-cli -a password --cluster fix xxx.xx.xxx.xx:6379`进行修复。

&nbsp;

## 三、异常与解决

#### SpringBoot使用redis集群出现的异常
- SpringBoot连接redis：
    - https://blog.csdn.net/lx1309244704/article/details/80696235
    - https://www.jianshu.com/p/b75e0d45b5e2

- **异常1**：在SpringBoot进行Redsi集群连接的时候报`ERR This instance has cluster support disabled`
- **解决**：Redis配置文件没有开启集群模式，在redis.conf中找到`cluster-enabled yes`并将其注释去掉。https://blog.csdn.net/z960339491/article/details/80521851

- **异常2**：SpringBoot2.x连接redis集群时出现`CLUSTERDOWN The cluster is down`或者`Node .. is not empty.Either the node already knows other nodes (check with CLUSTER NODES) or contains some key in database 0.`，一般出现在创建过集群然后重新创建的时候报的异常
- **解决**：需要删除redis的src目录下的`nodes.conf`文件，然后重新创建集群。

&nbsp;

#### Linux服务器搭建redis集群时出现的异常
- **异常1**：使用SpringBoot2.x（lettuce redis客户端）整合redis，报异常：`can not connect to 127.0.0.1：6379...`，连接不上集群
- **解决**：在进行集群创建的时候，使用`./redis-cli -a password --cluster create xxx.xx.xxx.xx:6379 xxx.xx.xxx.xx:6380 xxx.xx.xxx.xx:6381 xxx.xx.xxx.xx:6382 xxx.xx.xxx.xx:6383 xxx.xx.xxx.xx:6384 --cluster-replicas 1`进行创建的时候，需要用其`公网IP`进行创建

- **异常2**：进行redis集群创建的时候（公网IP创建），报`Waiting for the cluster to join ...`
- **解决**：开放其集群通讯端口，如集群使用的是6379-6384，则还需要再开放16379-16384端口，在阿里云防火墙进行开放，否则会一直无法创建--https://blog.csdn.net/Truong/article/details/52531103

- **异常3**：使用`redis-cli --cluster   check 127.0.0.1:6379`进行集群检查后，检查到`[ERR] Not all 16384 slots are covered by nodes.`（可能出现在集群有所更换之后）
- **解决**：在redis的src目录下运行` ./redis-cli -a password --cluster fix 172.168.63.201:7001`--https://blog.csdn.net/vtopqx/article/details/50235891 （redis5.0后使用redis-cli --cluster   check 127.0.0.1:6379等）

- **异常4**：配置从节点redis的`redis.conf`文件的`replicaof`指令后创建集群，报`replicaof directive not allowed in cluster mode`异常
- **解决**：如果是创建集群的话，需要注释掉`replicaof`的配置（本人为replica配置处配置了redis密码，这似乎使得主从复制成功了）

&nbsp;

#### 参考文献
- redis集群相关指令-1： https://www.cnblogs.com/ivictor/p/9768010.html

- redis集群相关指令-2： http://weizijun.cn/2016/01/08/redis%20cluster%E7%AE%A1%E7%90%86%E5%B7%A5%E5%85%B7redis-trib-rb%E8%AF%A6%E8%A7%A3/

- redis集群详解： http://russellluo.com/2018/07/redis-replication-demystified.html

&nbsp;

