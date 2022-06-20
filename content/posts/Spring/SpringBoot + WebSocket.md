+++

author = "pikachu"
title = "SpringBoot整合redis"
date = "2019-01-25"
description = " "
tags = [
    "",
    "spring",
	"redis",

]

categories = [
    "it", "spring"

]

+++



## SpringBoot整合WebSocket

- https://blog.csdn.net/moshowgame/article/details/80275084



整合websocket可能遇到的问题：

- 功能上：可行

  - 什么时候主动推送事件？如果是有事件进来的时候推送，是触发的时候去数据库（存在事件还没有插入数据库的问题）？如果不是查数据库而是直接推给前端，则只能单独发送一条，所以未读信息需要前端维护。

- 技术上：

  - 如何经过网关建立websocket？

    - https://cloud.spring.io/spring-cloud-gateway/reference/html/
    - 解决：直接可以通过spring cloud gateway负载到服务器上，看官方文档非常重要！找了一堆技术文章，还不如直接去看gateway官方文档，立马解决。

    ```
    service:
      uri:
    	ws-svc: lb:ws://ws-service
    
    @Value("${service.uri.ws-svc}")
    private String websocketSvcUri;
    
    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
    
            return builder.routes()
                    .route(r -> r.path("/ws-svc/**")
                            .filters(f -> f.rewritePath("/ws-svc/(?<path>.*)","/${path}"))
                            .uri(websocketSvcUri)
                    )
                    .build();
        }
    ```

    

  - 单体服务直接连接某台服务器的websocket，分布式环境下会导致服务的其他节点无法连接websocket，从而导致消息丢失。

    - 解决：独立部署ws服务（否则每次新建ws，都得更改gateway route，而且这样方便ws维护和横向扩展），由ws服务跟客户端建立连接并通过dubbo暴露消息发送的方法，由其他服务调用ws进行主动发送，解决服务多实例问题（dubbo调用也可以用MQ替换）

  - 如果websocket遇到性能瓶颈，横向扩展后如何共享ws session？

    - https://juejin.cn/post/6844903584929153032
    - 解决：