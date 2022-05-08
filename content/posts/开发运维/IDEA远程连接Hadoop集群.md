+++
author = "pikachu"
title = "IDEA远程连接Hadoop集群"
date = "2019-03-17"
description = " "
draft = false
tags = [
    "大数据","开发工具"
]
categories = [
    "it", "开发运维"
]

+++


## 环境
- **服务器**：CentOS 7
- **Hadoop版本**：2.7.7
- **cli端**：Windows 10
- **开发工具**：intellij IDEA

<br>

## 连接前提
- 已成功创建Hadoop集群，即能正常访问hdfs和mapreduce的web界面（本人是在虚拟机建的集群）
- 创建Maven项目并新建java  class WordCount（此处以单词统计为例子）

<br>

## Window环境准备
- 下载hadoop压缩包并解压到Window下（版本为集群所用的hadoop版本）
    - 下载地址：https://hadoop.apache.org/releases.html
    
- 下载winutils-master（Windows环境需要），解压后将其中对应版本的`bin`目录下的文件复制到`第一步解压的hadoop文件的bin目录下`（覆盖原有的）
    - 下载地址：http://www.pc0359.cn/downinfo/92994.html

- 将`winutils-master`中的`hadoop.dll`文件复制到Windows的`C:\Windows\System32`目录下
    - 参考：https://blog.csdn.net/zimojiang/article/details/80473201

- 配置Window环境变量
    - 在系统环境变量中加入：`HADOOP_HOME=D:\yangjm\Code\study\hadoop\hadoop-2.6.0`
    - 在Path中加入：`%HADOOP_HOME%\bin`

<br>

## IDEA相关准备
- pom.xml中加入相关依赖
```
   <dependencies>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-hdfs</artifactId>
            <version>2.7.7</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-client</artifactId>
            <version>2.7.7</version>
        </dependency>
    </dependencies>
```

- 将搭建好的hadoop集群的配置文件`log4j.properties`和`core-site.xml`复制后粘贴到IDEA创建的Java项目的resource文件夹下

- 为项目导入本地hadoop的相关jar包（下面图片来自腾讯云社区）
    - 打开Moudle设置
    
    ![51eg4f4syf](https://user-images.githubusercontent.com/38284818/54491462-c626d100-48f9-11e9-9143-b160e69421b2.png)

    - 导入hadoop相关jar包（在hadoop的`\share\hadoop\common`目录下）
    ![vbu5uan13q](https://user-images.githubusercontent.com/38284818/54491471-cc1cb200-48f9-11e9-827a-0146d01dede8.jpg)
    ![uz93dmlsxx](https://user-images.githubusercontent.com/38284818/54491475-cfb03900-48f9-11e9-90b4-e5ac094fd299.png)

    - 给导入的lib取个名字，如`hadoop`等
    ![ezmgol3yz4](https://user-images.githubusercontent.com/38284818/54491477-d048cf80-48f9-11e9-8994-3a9d11974da6.png)

- 设置运行参数：
    - 以空格隔开，第一个为hdfs中需要处理的文件，第二个为处理结果保存在hdfs的路径（hdfs地址在`core-site.xml`中）
    ![捕获](https://user-images.githubusercontent.com/38284818/54491663-1c484400-48fb-11e9-9010-22eb96d7ad91.JPG)

- 以上配置完后基本可以正常运行

<br>

## 可能出现的Bug
- 运行时出现权限问题
    - 解决：在JAVA代码中修改为有高级权限的用户
    - 参考：https://blog.csdn.net/diqijiederizi/article/details/82753573

<br>

## 参考
- https://blog.csdn.net/mm_bit/article/details/52118904
- https://cloud.tencent.com/developer/article/1024534