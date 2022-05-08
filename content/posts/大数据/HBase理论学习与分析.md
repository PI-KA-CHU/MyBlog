+++
author = "pikachu"
title = "HBase理论学习与分析"
date = "2019-03-22"
description = " "
draft = false
tags = [
    "大数据"
]
categories = [
    "it"
]

+++



<br>

## Hbase简介

- HBase利用Hadoop的**HDFS**作为其文件存储系统，提供**高可靠性、高性能、列存储，可伸缩、实时读写**的**分布式的**数据库系统，利用Zookeeper作为其协同服务，适合于**非结构化数据存储**的分布式数据库。

![image](https://user-images.githubusercontent.com/38284818/54814276-57b38b80-4cca-11e9-9854-823ed7c309ff.png)

<br>

## Hbase特点

- **存储特点**：

    - **面向列存储**
    - **存储量大**：一个表可以有上亿行，上百万列（列多时，插入变慢）
    - **数据稀疏**：对于空（null）的列并不占用存储空间，因此，表可以设计的非常稀疏
        - 补充：**稀疏数据**是指在数据集中绝大多数数值缺失或者为零的数据
    - **数据类型单一**：HBase中的数据都是**字符串**，没有类型

<br>

- **表的构造**：

    - 每张表里面有多个rowkey（行键），每个rowkey包含多个columnFamily（列族），每个coloumnFamily由一个或者多个cloumn（列）组成。
    - 数据查询结果：`Row key + column family:column + timestamp + value`

<br>

- **传统关系型数据库和列式数据库的区别**：

![image](https://user-images.githubusercontent.com/38284818/54814247-49656f80-4cca-11e9-8a44-289b4c262872.png)

<br>

## HBase的原理

- **HBase的属性**

    - **Row Key**
        - 表中每条记录的“主键”
        - 方便快速查找
    - **Column Family**
        - 拥有一个名称(string)
        - 包含一个或者多个相关列
    - **Column**
        - 属于某一个Column Family
        - 包含在某一列中
    - **Timestamp**
        - 每个rowKey有唯一一个
        - 类型为long
        - 默认值是系统时间戳
    - **Value**
        - 某一列属性的值（String）

<br>

- **HBase的存储**

    - https://www.cnblogs.com/duanxz/p/3154487.html
    - Table中的所有行都按照**row key**的**字典序排列**，类型为**byte字节流**，Table在行的方向上分割为多个**region**；
    ![image](https://user-images.githubusercontent.com/38284818/54820968-cc42f600-4cdb-11e9-90e9-88ec428aae5f.png)
    
    <br>
    
    - region是**按照大小分割**的，每个表开始只是一个region，随着数据的增多，region不断增大，当达到一个值的时候会**等分**成两个region。
     ![image](https://user-images.githubusercontent.com/38284818/54821572-9141c200-4cdd-11e9-9b8a-d941ab550191.png)
    
    <br>
    
    - region是**分布式存储**的最小单元，但并不是存储的最小单元。
    
        - region由一个或者多个**Store**组成，每个Store保存一个columns family；
        
        - 每个store由一个**MemStore**和**多个StoreFIle**组成组成；
        
        - **MemStore存储在内存中，StoreFile存储在hdfs上**；
            - 首先数据存储到MemStore上，当数据量达到一定的大小，再flush到storefile上，形成一个StoreFile。
            ![image](https://user-images.githubusercontent.com/38284818/54822095-22656880-4cdf-11e9-8bb4-8d77f0ab0f6e.png)
  
    <br>
    
- **HBase支持的操作**
    - 所有操作都是基于row key的；

    - 支持CRUD（**create、read、update、delete**）和**scan**
        - 单行操作：**put、get、scan**
        - 多行操作：**scan、multiPut**
    - **没有内置join操作**，需要使用mapreduce解决    
    

<br>

## HBase的架构

- **架构图**
 ![image](https://user-images.githubusercontent.com/38284818/54823588-4e82e880-4ce3-11e9-802f-7f8e5a98b0ab.png)
   
    <br>
   
- **架构角色分析**

    - **Client**：包含访问HBase的接口，并维护**cache**来加快对HBase的访问（即缓存遍历hbase:meta的区域数据）
    
    - **Zookeeper**：
    
        - 保证任何时候，集群中只有一个**master**
        - 存储【所有region的寻址入口】（即`hbase:meta`的位置）
            - hbase:mate：维护着当前集群上所有区域的**列表、状态和位置**
        - **实时监控`RegionServer`的上线和下线信息**，并实时通知给Master存储HBase的schema和table的**元数据**。
    
    - **Master**：
      
        - 为Region server **分配region**
        - 负责Region server的**负载均衡**
        - 发现失效的Region server并**重新分配**其上的region
        - 管理用户对**table的增删该查**操作
    
    - **Region Server**：
    
        - 维护region，**处理这些region的IO请求**
        - 负责**切分**在运行过程中变得过大的region

<br>

- **HBase的容错性**

    - Master容错：Zookeeper重新选择一个新的Master
    
        - 无Master过程中：数据读取仍照常进行
        - 无Master过程中：region切分、负载均衡等无法进行
        
    - RegionServer容错：**定时的向Zookeeper汇报心跳**，如果一段时间内未出现心跳，Master将会将该RegionServer的Region **重新分配**到其他RegionServer上。
        - 失效服务器上的**“预写”日志**由主服务器进行分割并派送给**其他RegionServer**。


    - Zookeeper容错：Zookeeper是一个**可靠地服务**，一般配置3或5个Zookeeper实例。

<br>

## 分区查看

- 进入HBase Master主页`http://hh:60030/rs-status`可以查看
    - MapReduce如果以HBase为源输入，其Map数量由`Region数量`决定，如果源输入是HDFS，其Map数量是由`HDFS文件分片`决定。

![image](https://user-images.githubusercontent.com/38284818/62531923-fa356b00-b875-11e9-8292-b26eee9316c5.png)


<br>

## 参考
- 整理自： http://wenku.uml.com.cn/document.asp?fileid=17427&partname=%B4%F3%CA%FD%BE%DD
