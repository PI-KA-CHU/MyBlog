+++

author = "pikachu"
title = "SpringBoot整合redis"
date = "2022-06-28"
description = " 微服务下整合websocket"
tags = [
    "websocket",
    "spring-cloud", "分布式",

]

categories = [
    "it", "spring"

]

+++



## SpringCloud整合WebSocket

- https://blog.csdn.net/moshowgame/article/details/80275084



## 个人探索版

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
    - 解决：通过redis进行session共享，但是ws session网上说是有序列化问题，我还需要验证一下。
  
  



## 同事版

> 相比于我个人探索的版本，同事这边的代码就比较完善，利用到了拦截器并做了分组件封装，而且他发送websocket的思路不太一样，我原本是想每个需要主动推送的地方都需要有单独的websocket endpoint，而他是只用一个endpoint去发送多个地方的websocket，通过一个字段进行确定这个websocket是显示在哪个模块的，好处是代码量减少，代码通用性变强，对于同个用户，可以避免创建大量websocket连接。



**HttpAuthHandler**

> 该类作为建立连接、断开连接、发送文本时的钩子

```
@Component
public class HttpAuthHandler extends TextWebSocketHandler {
    static Logger logger = LoggerFactory.getLogger(HttpAuthHandler.class);
    @Override
    public void afterConnectionEstablished(WebSocketSession session) throws Exception {

        Object token = session.getAttributes().get("token");
        if (token != null) {
            // 用户连接成功，放入在线用户缓存
            WsSessionManager.add(token.toString(), session);
        } else {
            throw new RuntimeException("用户登录已经失效!");
        }
    }

    @Override
    protected void handleTextMessage(WebSocketSession session, TextMessage message) throws Exception {
        String payload = message.getPayload();
        Object token = session.getAttributes().get("token");
        logger.info("server 接收到 " + token + " 发送的 " + payload);
        session.sendMessage(new TextMessage("server 发送给 " + token + " 消息 " + payload + " " + LocalDateTime.now().toString()));
    }

    @Override
    public void afterConnectionClosed(WebSocketSession session, CloseStatus status) throws Exception {
        Object token = session.getAttributes().get("token");
        if (token != null) {
            // 用户退出，移除缓存
            logger.info("websocket 用户退出:" + token);
            WsSessionManager.remove(token.toString());
        }
    }
}

```



**MyInterceptor**

> 该类作为websocket握手前和握手后的钩子

```
@Component
public class MyInterceptor implements HandshakeInterceptor {

    static Logger logger = LoggerFactory.getLogger(MyInterceptor.class);

    @Override
    public boolean beforeHandshake(ServerHttpRequest serverHttpRequest, ServerHttpResponse serverHttpResponse, WebSocketHandler webSocketHandler, Map<String, Object> map) throws Exception {
        String hostName = serverHttpRequest.getLocalAddress().getHostName();
        logger.info("websocket 开始握手" + hostName);
        String uuid = UUID.randomUUID().toString();
        logger.info("随机uuid:"+uuid);
        map.put("token",uuid);
        return true;
    }

    @Override
    public void afterHandshake(ServerHttpRequest serverHttpRequest, ServerHttpResponse serverHttpResponse, WebSocketHandler webSocketHandler, Exception e) {
        logger.info("握手完成");

    }
}
```



**WebSocketConfig**

> 该类作为websocket的核心配置类，注册websocket endpoint并注入前面的两个钩子类

```
@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {

    @Autowired
    private HttpAuthHandler httpAuthHandler;

    @Autowired
    private MyInterceptor myInterceptor;

    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry webSocketHandlerRegistry) {
        webSocketHandlerRegistry
                .addHandler(httpAuthHandler,"/websocket")
                .addInterceptors(myInterceptor)
                .setAllowedOrigins("*");
    }
}
```



**WsSessionManager**

> 该类用于管理websocket的session，以及向所有session连接的用户发送消息

```
public class WsSessionManager {
    private static ConcurrentHashMap<String, WebSocketSession> SESSION_POOL = new ConcurrentHashMap<>();

    /**
     * 添加 session
     *
     * @param key
     */
    public static void add(String key, WebSocketSession session) {
        // 添加 session
        SESSION_POOL.put(key, session);
    }

    /**
     * 删除 session,会返回删除的 session
     *
     * @param key
     * @return
     */
    public static WebSocketSession remove(String key) {
        // 删除 session
        return SESSION_POOL.remove(key);
    }

    /**
     * 删除并同步关闭连接
     *
     * @param key
     */
    public static void removeAndClose(String key) {
        WebSocketSession session = remove(key);
        if (session != null) {
            try {
                // 关闭连接
                session.close();
            } catch (IOException e) {
                // todo: 关闭出现异常处理
                e.printStackTrace();
            }
        }
    }

    /**
     * 获得 session
     *
     * @param key
     * @return
     */
    public static WebSocketSession get(String key) {
        // 获得 session
        return SESSION_POOL.get(key);
    }

    public synchronized static void sendMessage(String json) {
        TextMessage textMessage = new TextMessage(json);
        try {
            for (WebSocketSession session : SESSION_POOL.values()) {
                session.sendMessage(textMessage);
            }
        }catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

