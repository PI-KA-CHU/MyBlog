 +++
author = "pikachu"
title = "HBase的搭建及连接（IDEA）"
date = "2019-03-29"
description = "Lorem Ipsum Dolor Si Amet"
draft = false
tags = [
    "大数据"
]
categories = [
    "IT"
]
+++



## 搭建参考
- 参考： https://www.cnblogs.com/edisonchou/p/4405906.html
- 根据这篇文章可以正常搭建平台，HBase的配置与Hadoop类似，分别修改`hbase-env.sh`和`hbase-site.xml`即可（Linux环境下搭建）
- Window客户端（IDEA）连接HBase，需要导入`HBase下的lib目录`，可参考
- 将搭建好的HBase的`hbase-site.xml`文件复制粘贴到项目的`resource`目录下，跟搭建hadoop时候类似

## Bug
- `java.net.UnknownHostException: unknown host: node-1`异常
    - **解决**：需要修改Window下的地址映射，在`C:\WINDOWS\system32\drivers\etc\hosts`文件中添加如下信息：`192.125.168.135 node-1`（地址对应虚拟机HBase节点所在主机地址）
    - **参考**： https://blog.csdn.net/lifuxiangcaohui/article/details/40861079
