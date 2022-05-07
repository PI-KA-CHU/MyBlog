 +++
author = "pikachu"
title = "Flume收集日志 - windows（配置版）"
date = "2019-04-10"
description = "Lorem Ipsum Dolor Si Amet"
draft = false
tags = [
    "大数据"
]
categories = [
    "IT"
]
+++



### Flume的一些常见参数：
- https://blog.csdn.net/realoyou/article/details/81514128

### Windows配置Flume
- https://www.cnblogs.com/edisonchou/p/4445491.html
- https://blog.csdn.net/wateryouyo/article/details/82057082
- https://blog.csdn.net/woshixiazaizhe/article/details/80605293
- https://www.cnblogs.com/chevin/p/8491721.html

### windows环境下使用flume的权限问题：
- http://www.huqiwen.com/2013/07/18/hdfs-permission-denied/

### commons-io缺包问题：
- http://mangocool.com/1464850911377.html

### event未捕获到时间戳问题
- http://blog.sina.com.cn/s/blog_db77b3c60102vrzt.html

### hadoop缺包问题
- https://www.cnblogs.com/zhutianye/articles/5012346.html

### flume上传到hdfs出现大量小文件问题（minBlockReplicas参数的配置）
- https://blog.csdn.net/whdxjbw/article/details/80606917

### JAVA程序log4j整合flume实现日志上传到HDFS
- https://blog.csdn.net/hzs33/article/details/79429087
- https://blog.csdn.net/antgan/article/details/52087926

### Flume分层收集日志
- https://blog.csdn.net/lbship/article/details/84951829

### Flume异常及解决：

- **异常**：`java.nio.charset.MalformedInputException: Input length = 1 `

- **解决**：编码设置问题：https://blog.csdn.net/qq_32967001/article/details/66973094

- **异常**：`java.io.IOException: Could not locate executable null\bin\winutils.exe in the Hadoop binaries.`

- **解决**：hadoop环境变量未配置问题：https://blog.csdn.net/baidu_19473529/article/details/54693523

