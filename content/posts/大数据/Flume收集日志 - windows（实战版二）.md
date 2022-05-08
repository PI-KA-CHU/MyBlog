+++
author = "pikachu"
title = "Flume收集日志 - windows（实战版二）"
date = "2019-05-22"
description = " "
draft = false
tags = [
    "大数据"
]
categories = [
    "it"
]

+++


### 一、写在前面
Flume在Windows上的使用还真是各种不方便，但是没办法，项目就是搭建在Windows服务器上的，前面的文章是使用自定义拦截器解决了日期划分的问题，接下来解决的是实时监控日志更新的问题

&nbsp;

### 二、Flume的三种Source
- **Exec Source**：
    可通过`tail -f`命令去tail住一个文件，然后实时同步日志到sink。但存在的问题是，当agent进程挂掉重启后，**会有重复消费的问题**。可以通过增加UUID来解决，或通过改进ExecSource来解决。

- **Spooling Directory Source**：
    监视一个目录，同步目录中的新文件到sink，同步完的文件会打上**后缀标识符号**（默认为.COMPLETE）,适用于同步新文件，但是**不适用于对实时追加日志文件的监督**。上一篇文章就是用Spooldir source，但是日志是实时追加的，监控时会因为无法关闭而报异常。

- **Taildir Source**：
    - 1.7版flume新加入的source，解决了**多个文件夹监督**的问题，同时可以通过目录直接用正则表达式直接监督符合的文件，而且**不会存在重复消费的问题**，数据已经读取的位置信息被保存，在另一个文件中，所以即使完成也不会为文件加后缀，可**实时监控多个文件**。本编文章使用的就是Taildir source（真的好用）。
    - 为了**唯一标示**一个文件，该source利用**操作系统inode**的方式获得**文件的一个id**，目前仅采用unix的方式获取，不支持window，需要更改源码添加window获取inode的方法

#### Taildir source在Windows环境下的使用

- flume虽然强大，但是有点坑爹的是不太兼容Windows平台，比如Taildir Source源码中在inode的处理上是在Linux环境下实现的，如果要在Windows上使用需要重新修改并编译源码，下面讲一下自己的一个编译过程（**具体流程按照下面参考文章即可成功**）。

- **参考**： https://www.jianshu.com/p/5a53c002b1dd

&nbsp;

### 三、问题及解决

1. 下载flume源码后，使用编译器打开项目（本人使用的是IDEA，以maven项目的形式导入）

2. 导入项目后可以只下载`flume-ng-source`的maven依赖包即可，就是说不用整个项目都编译成功，可以通过cmd进入该项目下的`flume-ng-source`根目录，将修改完的项目（按照上面参考）进行编译即可。
![image](https://user-images.githubusercontent.com/38284818/58153592-03189680-7ca2-11e9-9e80-a56d7afc3183.png)

3. maven编译时可能会出现下面格式异常，使用编译命令`mvn clean install -Dcheckstyle.skip=true`即可解决。
![image](https://user-images.githubusercontent.com/38284818/58153647-26434600-7ca2-11e9-8640-a4f152bdbec8.png)

4. flume传输文件到HDFS后，HDFS中保存的文件有`.tmp`的后缀，原因是文件仍被打开着，即仍被flume监视及占用着，此时存储在的字节大小是无法正常读取到的，解决方法（**flume配置**）：

```
//18000秒内如果文件没有被追加上传，则flume关闭文件，此时HDFS文件可以显示正常文件大小
a1.sinks.k2.hdfs.idleTimeout = 18000
```
- 参考： https://bit1129.iteye.com/blog/2186026