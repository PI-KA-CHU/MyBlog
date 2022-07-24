

---

title = "Netty实战"
date = "2022-07-24"
description = " 简单netty通信案例 "
draft = false
tags = [
    "netty"
]
categories = [
    "it", "中间件"
]

---

## Netty实战

### 前言

> 这几天进行了情报部通信相关的开发，想要Netty加入到开发过程中，在整合到过程出现了一些难以解决的问题，情报板发送分为四个阶段：
>
> 1. 发送删除旧的情报板文件请求
> 2. 发送创建新的文件请求
> 3. 发送文件内容
> 4. 发送播放文件的请求
>
> 按照传统的IO模型，我们可以每次发送socket后阻塞等待返回，以确定是否进行下一步操作，但是采用Netty后我就有点懵了，因为Netty本身定位为非阻塞型IO，不知道如何进行阻塞某一个的结果并根据结果确定下一步执行，本次Demo通过Netty的Handler + Promise实现上述内容。
>
> 原来的情报板实现发送的逻辑如下：
>
> 1. 接收到Kafka消息后，启动新线程去执行情报板发送
> 2. 在发送情报板的Service类中，BIO Socket作为static静态变量进行通信（这意味着多线程下会共享同个Socket，数据的写入将会出现无法预期的混乱情况（并发写入Socket），并且发布情报板的四个阶段也会因为并发操作而出现问题。）
>
> 解决方案：
>
> 1. 采用Netty NIO框架
> 2. 单线程池替换new Thread的方式 -> 采用可缓存线程池（多线程还是需要开启，因为对多个情报板同时更新是完全没问题的，单线程无法保证其最大并发）
> 3. 对情报板的四步操作加锁保证其原子性，并且为了提高并发，降低锁的粒度，对设备IP进行加锁，即保证同个情报板的更新是同步的。
>
> 疑惑：
>
> 1. Netty的NIO在代码层面到底具体在哪部分？
>
> 目前的猜测，传统的IO操作，我们直接对socket操作，并耦合业务代码，这导致我们的工作线程需要阻塞等待IO，而Netty的业务我们可以封装到handler中，通过channel异步写入数据后，后续的业务操作由Netty专有线程池控制Handler进行操作，当前工作线程可以返回。由于情报板的更新有序性及操作原子性特性，所以需要对情报板加锁处理，保证当前操作完成后进行下一次的更新。



### 依赖

- 该springboot依赖包含netty-all依赖包

```
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
```



### 客户端

**1. 客户端服务启动类**

```java
public class NettyClient {

    public static void start() throws InterruptedException {
        NioEventLoopGroup nioEventLoopGroup = new NioEventLoopGroup();

        // promise用于获取业务的执行结果
        EventExecutor eventExecutor = new DefaultEventExecutor();
        Promise<Integer> promise = new DefaultPromise<>(eventExecutor);

        Bootstrap bootstrap = new Bootstrap();
        bootstrap
                .group(nioEventLoopGroup)
                .channel(NioSocketChannel.class)
                .option(ChannelOption.SO_KEEPALIVE, true)
                .option(ChannelOption.TCP_NODELAY, true)
                .handler(new ChannelInitializer<SocketChannel>() {
                    @Override
                    protected void initChannel(SocketChannel socketChannel) throws Exception {
                        ChannelPipeline pipeline = socketChannel.pipeline();
                        pipeline.addLast(new ClientDecoder());
                        pipeline.addLast(new ClientByteEncoder());
                        pipeline.addLast(new ClientHandler(promise));
                    }
                });

        // channel无法序列化，所以不能直接传输
        Channel channel = bootstrap.connect("127.0.0.1", 8090).await().channel();

        OperateDto operateDto = OperateDto.builder()
                .operate("1")
                .content("这是client的1")
                .build();
        channel.writeAndFlush(operateDto);

        System.out.println("发送内容：" + operateDto);

        try {
            Integer integer = promise.await().get();

            System.out.println("最终执行结果：" + integer);
        } catch (ExecutionException e) {
            System.out.println("最终执行结果：" + e.getCause().getMessage());
        }

    }
}
```



2. **客户端Handler处理类**

- **编码类**：用于做客户端的编码工作，通常协议的构建就是在此类中进行，然后发送给服务端

```java
public class ClientByteEncoder extends MessageToByteEncoder<OperateDto> {

    @Override
    protected void encode(ChannelHandlerContext channelHandlerContext, OperateDto operateDto, ByteBuf byteBuf) throws Exception {
        byte[] serialize = SerializationUtils.serialize(operateDto);
        System.out.println("客户端encoder发送：" + Arrays.toString(serialize));
        byteBuf.writeBytes(serialize);
    }
}
```

- **解码类**：用于接收和解析从服务端发送来的数据，校验及内容的提取就是在此类进行

```java
public class ClientDecoder extends ByteToMessageDecoder {
  	
	  // 第二个参数为in（from server），第三个参数为out（out to next handler）
    @Override
    protected void decode(ChannelHandlerContext channelHandlerContext, ByteBuf byteBuf, List<Object> list) throws Exception {
      	// ByteBuf byteBuf = channelHandlerContext.alloc().buffer();
      
        // byteBuf用于存储通信内容
        byte[] resp = new byte[byteBuf.readableBytes()];
        // 将byteBuf的内容读取到resp数组中
        byteBuf.readBytes(resp);
        // 将读取到到byte数组进行反序列化操作
        ResultDto resultDto = (ResultDto) SerializationUtils.deserialize(resp);
        System.out.println("接收到服务器响应：" + resultDto);

        // list即为out，加入后会传送给下一个handler
        list.add(resultDto);
    }
}
```

- **业务处理类**：该类可以认为是Decoder类的下一层解析，Decoder解析协议并提取出内容，Handler将内容进行业务层相关的解析工作，并且只有当监听到相同类型的对象时才会有效，否则会忽略，例如下面的handler，需要监听到上游handler发送了ResultDto类型的数据才会响应。

```java
public class ClientHandler extends SimpleChannelInboundHandler<ResultDto> {

    /**
     * 接收来自客户端的promise，当业务达到结束条件后，setSuccess或者setFailure，
     * 客户端监听到结果后后进行相关处理，例如通过addListening异步监听，或者通过sync同步阻塞等待结果
     */
    private final Promise<Integer> promise;

    public ClientHandler(Promise<Integer> promise) {
        this.promise = promise;
    }

    /**
     *  将channel和promise的操作放在handler中，与service解耦
     *  promise：用于确定业务的执行结果，并将结果返回到client
     *  channel：用于服务端和客户端间到通信，向channel写入数据即可向对方通信
     *  ByteBuf：用于存储接收到的通信内容，通常需要从中读取内容并进行接下来的业务操作
     */
    @Override
    protected void channelRead0(ChannelHandlerContext channelHandlerContext, ResultDto resultDto) throws Exception {
        ClientService clientService = SpringContextUtil.getBean(ClientService.class);
        Channel channel = channelHandlerContext.channel();

        // 将业务逻辑抽离到service层，获取业务执行结果
        OperateDto operateDto = clientService.handleResult(resultDto);
        channel.writeAndFlush(operateDto);

        // 根据结果判定是否达到业务结束条件，将结果添加到promise中
        String operate = operateDto.getOperate();
        if ("3".equals(operate)) {
            promise.setSuccess(10000);
//            promise.setFailure(new Exception("出错啦"));
        }

    }
}
```



**3. 客户端Service层业务处理**

- 抽离出来的业务执行类，输入是服务器的响应内容，输出是处理后接下来的执行内容

```java
@Service
public class ClientService {

    public OperateDto handleResult(ResultDto resultDto) {
        String flag = resultDto.getFlag();
        OperateDto operateDto = OperateDto.builder().build();

        switch (flag) {
            case "1":
                operateDto.setOperate("2");
                operateDto.setContent("接收到1，开始执行2");
                break;
            case "2":
                operateDto.setOperate("3");
                operateDto.setContent("接收到2，开始执行3");
                break;
        }

        return operateDto;
    }
}
```



### 服务端

1. **服务端启动类**

```
public class NettyServer {

    public static void start() {
        NioEventLoopGroup bossExecutors = new NioEventLoopGroup();
        NioEventLoopGroup workerExecutors = new NioEventLoopGroup();

        ServerBootstrap serverBootstrap = new ServerBootstrap();
        serverBootstrap
                .group(bossExecutors, workerExecutors)
                .channel(NioServerSocketChannel.class)
                .childOption(ChannelOption.SO_KEEPALIVE, true)
                .childOption(ChannelOption.TCP_NODELAY, true)
                .childHandler(new ChannelInitializer<SocketChannel>() {

                    @Override
                    protected void initChannel(SocketChannel socketChannel) throws Exception {
                        ChannelPipeline pipeline = socketChannel.pipeline();
                        pipeline.addLast(new ServerByteEncoder());
                        pipeline.addLast(new ServerByteDecoder());
                        pipeline.addLast(new ServerHandler());
                    }
                });

        serverBootstrap.bind(8090);
    }
}
```



**2. 服务端Handler处理类**

- **服务端解码类**

```
public class ServerByteDecoder extends ByteToMessageDecoder {

    @Override
    protected void decode(ChannelHandlerContext channelHandlerContext, ByteBuf byteBuf, List<Object> list) throws Exception {
        byte[] con = new byte[byteBuf.readableBytes()];
        byteBuf.readBytes(con);

        OperateDto operateDto = (OperateDto) SerializationUtils.deserialize(con);
        System.out.println("服务端decoder收到消息：" + operateDto);
        list.add(operateDto);
    }
}
```

- **服务端编码类**

```
public class ServerByteEncoder extends MessageToByteEncoder<ResultDto> {

    @Override
    protected void encode(ChannelHandlerContext channelHandlerContext, ResultDto resultDto, ByteBuf byteBuf) throws Exception {

        System.out.println("服务端响应处理结果：" + resultDto);
        byteBuf.writeBytes(SerializationUtils.serialize(resultDto));
    }
}
```

- **服务端处理类**

```
public class ServerHandler extends SimpleChannelInboundHandler<OperateDto> {


    @Override
    protected void channelRead0(ChannelHandlerContext channelHandlerContext, OperateDto operateDto) {
        System.out.println("server handler 接收到：" + operateDto);

        ServerService serverService = SpringContextUtil.getBean(ServerService.class);
        Channel channel = channelHandlerContext.channel();

        ResultDto resultDto = serverService.operateClientReq(operateDto);

        // 服务端可以通过channel跟客户端进行通信，每个客户端与服务端的连接都会有一个channel
        channel.writeAndFlush(resultDto);
    }
}
```



**3. 服务端Service层业务处理**

```
@Service
public class ServerService {
    
    public ResultDto operateClientReq(OperateDto operateDto) {
        String operate = operateDto.getOperate();
        ResultDto resultDto = ResultDto.builder().build();

        switch (operate) {
            case "1":
                resultDto.setFlag("1");
                resultDto.setResult("这是server对你1的回复");
                break;
            case "2":
                resultDto.setFlag("2");
                resultDto.setResult("这是server回复你2的回复");
                break;
        }

        return resultDto;
    }
}
```



### 其他

- OperateDto

```
@Data
@Builder
@AllArgsConstructor
@NoArgsConstructor
public class OperateDto implements Serializable {
    private String operate;
    private String content;
}
```

- ResultDto

```
@Data
@Builder
@AllArgsConstructor
@NoArgsConstructor
public class ResultDto implements Serializable {

    private String flag;
    private String result;
}
```

- SpringContextUtil

```
@Component
public class SpringContextUtil implements ApplicationContextAware {

    private static ApplicationContext applicationContext;

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        SpringContextUtil.applicationContext = applicationContext;
    }

    //获取applicationContext
    public static ApplicationContext getApplicationContext() {
        return applicationContext;
    }

    //通过name获取 Bean.
    public static Object getBean(String name) {
        return getApplicationContext().getBean(name);
    }

    //通过class获取Bean.
    public static <T> T getBean(Class<T> clazz) {
        return getApplicationContext().getBean(clazz);
    }

    //通过name,以及Clazz返回指定的Bean
    public static <T> T getBean(String name, Class<T> clazz) {
        return getApplicationContext().getBean(name, clazz);
    }

}
```

