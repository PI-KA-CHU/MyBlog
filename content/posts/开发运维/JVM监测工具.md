

## Arthas

- 文档：https://arthas.aliyun.com/doc/tunnel.html
- 常见命令：https://arthas.aliyun.com/doc/commands.html
- 工具实战：
  - https://juejin.cn/post/6844903745323565064



**依赖包**

```
<dependency>
    <groupId>com.taobao.arthas</groupId>
    <artifactId>arthas-spring-boot-starter</artifactId>
    <version>${arthas.version}</version>
</dependency>
```



### 单机版

- 访问：http://localhost:3658

```
arthas:
  tunnel-server: ws://127.0.0.1:7777/ws
  app-name: event-management-service

management:
  endpoints:
    web:
      exposure:
        include: '*'
  endpoint:
    health:
      show-details: always
```



### 微服务版

1. 下载并部署arthas tunnel server

   - 下载：https://github.com/alibaba/arthas/releases

   - 启动：`java -jar  arthas-tunnel-server.jar`启动服务

   - 访问： http://localhost:8080/apps.html 即可看到所有注册的arthas列表

     ![image-20220727162134132](https://raw.githubusercontent.com/PI-KA-CHU/Image-OSS/main/images/image-20220727162134132.png)

2. 其他服务中引入依赖后，加入下列配置（本质为注入服务到arthas tunnel server）

```
arthas:
  tunnel-server: ws://127.0.0.1:7777/ws
  app-name: event-management-service
  http-port: -1  // -1为关闭监听
  telnet-port: -1
```







## VisualVM



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

- 原因：接口请求一直处于pending状态，以为是JVM出了问题，接入VisualVM后发现运行正常

- 解决：后来在error日志中发现是redis连接错误，需要注意error日志，在out日志并不会打印错误，所以是因为reids一直阻塞导致的问题。