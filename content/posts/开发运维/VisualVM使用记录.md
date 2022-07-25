

**VisualVM远程连接**：

- https://www.jianshu.com/p/6d19e83aa1ea

1. 远程服务启动时添加以下参数

```
-Dcom.sun.management.jmxremote=true 
-Dcom.sun.management.jmxremote.port=33306 
-Dcom.sun.management.jmxremote.authenticate=false 
-Dcom.sun.management.jmxremote.ssl=false
```

2. 添加JMX远程连接

   ![image-20220712182846848](https://raw.githubusercontent.com/PI-KA-CHU/Image-OSS/main/images/image-20220712182846848.png)

3. 确定连接

   ![image-20220712182947533](https://raw.githubusercontent.com/PI-KA-CHU/Image-OSS/main/images/image-20220712182947533.png)





GC记录

```
1. GC后MAX_HEAP增大，应该MAX和MIN没有设置成一样大小
```



问题记录：

接口请求一直处于pending状态，后来发现是redis连接错误，需要注意error日志，在out日志并不会打印错误，所以是因为reids一直阻塞导致的问题。