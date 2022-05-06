+++
author = "pikachu"
title = "SpringBoot整合redis"
date = "2019-01-25"
description = "Lorem Ipsum Dolor Si Amet"
tags = [
    "java",
    "spring",
	"中间件"
]
categories = [
    "IT"
]
+++


## 一、redis单机版

#### 写在前面
- 选中代码块，按`control + alt + t`：快捷**tryCatch**
- 选中类名，按`control + shift + t`：快捷创建**Junit Test**


#### redis连接的准备工作
- 如果是阿里云购买的服务器的话，需要**开放redis的启动端口**，默认端口为6379
- 打开redis根目录下的redis.conf文件，将`bind 127.0.0.1`注释掉，即改为`# bind 127.0.0.1`，否则只能进行本地连接，从而报异常
- 找到redis.conf文件中的`protected-mode yes`，将其改为`protected-mode no`，关闭保护模式，否则会报连接被终止的异常
- 重启redis，重启的使用应携带conf文件进行重启，在redis的src文件下执行`./redis-server ../redis.conf`重启redis，直接重启配置文件并不会生效



#### redis相关配置
- **依赖包引入**
```
<!-- 引入redis依赖 -->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>

<!-- redis依赖commons-pool 这个依赖一定要添加 -->
<dependency>
	<groupId>org.apache.commons</groupId>
	<artifactId>commons-pool2</artifactId>
</dependency>
```

- 配置springBoot的配置文件（application.properties或application.yml），此处配置的是**application.properties**
```
# redis相关配置
# redis数据库索引
spring.redis.database=0
# redis服务器地址
spring.redis.host=120.78.151.65
# redis服务器连接端口
spring.redis.port=6379
# redis服务器连接密码
spring.redis.password=
# 连接池最大连接数（负值表示没有限制）
spring.redis.lettuce.pool.max-active=8
# 连接池最大阻塞等待时间（负值表示无限制）
spring.redis.jedis.pool.max-wait=-1
# 连接池中的最大空闲连接
spring.redis.lettuce.pool.max-idle=8
# 连接池中的最小空闲连接
spring.redis.lettuce.pool.min-idle=0
# 连接超时时间（毫秒）
spring.redis.timeout=5000
```

- 创建`RedisConfig.java`文件，配置自定义**redisTemplate**，解决因默认序列化`JdkSerializationRedisSerializer`导致的字符显示不正常的问题。

```
import com.fasterxml.jackson.annotation.JsonAutoDetect;
import com.fasterxml.jackson.annotation.PropertyAccessor;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.boot.autoconfigure.AutoConfigureAfter;
import org.springframework.boot.autoconfigure.data.redis.RedisAutoConfiguration;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;

@Configuration
@AutoConfigureAfter(RedisAutoConfiguration.class)
public class RedisConfig {

    /**
     * 配置自定义redisTemplate
     * @return
     */
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {

        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(redisConnectionFactory);

        //使用Jackson2JsonRedisSerializer来序列化和反序列化redis的value值
        Jackson2JsonRedisSerializer serializer = new Jackson2JsonRedisSerializer(Object.class);

        ObjectMapper mapper = new ObjectMapper();
        mapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        mapper.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        serializer.setObjectMapper(mapper);

        template.setValueSerializer(serializer);
        //使用StringRedisSerializer来序列化和反序列化redis的key值
        template.setKeySerializer(new StringRedisSerializer());
        template.setHashKeySerializer(new StringRedisSerializer());
        template.setHashValueSerializer(serializer);
        template.afterPropertiesSet();
        return template;
    }

}
```

- 创建**Junit测试**实例进行Junit测试,Idea编译器中选中类名，`control + shift + t`快捷创建
```
import org.junit.Test;
import org.junit.runner.RunWith;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.test.context.junit4.SpringRunner;

import java.util.concurrent.TimeUnit;

@RunWith(SpringRunner.class)
@SpringBootTest
public class RedisConfigTest {

    private Logger log = LoggerFactory.getLogger(this.getClass());

    @Autowired
    private RedisTemplate redisTemplate;

    @Test
    public void redisTemplate() {

        // redis存储的键名
        String key = "number";

        //如果不存在则设置值，存在的话则不设置
        redisTemplate.opsForValue().setIfAbsent(key, 1);

        //设置哈希值map值
        redisTemplate.opsForHash().put("map", "place", "北京");
        //获取到哈希值
        String place = (String) redisTemplate.opsForHash().get("map","place");
        log.info("[获取到的哈希值] - {}",place);

        //进行值的增加和减少（需要为Integer）
        redisTemplate.opsForValue().increment("number",-10);
        
        //log中的“{}”为占位符
        String value = (String) redisTemplate.opsForValue().get("school");
        log.info("[redis中 获取到的值] - [{}]",value);

        //设置有效时间为10秒的键值
        redisTemplate.opsForValue().set("flag","10",10, TimeUnit.SECONDS);
        String flag = (String) redisTemplate.opsForValue().get("flag");
        log.info("[获取到的有效时间为10秒的flag] - [{}]",flag);
    }
}
```

&nbsp;

#### Redis整合遇到的坑
**问题一：**
- SpringBoot整合Redis后连接Redsi出现**超时**问题

**解决：**
- 网上的redis的相关配置中都是`spring.redis.timeout=0`，超时时间不能设置为0，可以设置为5000（根据实际情况设置）。

**问题二：**
- 修改redis的conf文件后重启redis配置没有生效，一直报连接被主机中的软件中止。
```
org.springframework.data.redis.RedisSystemException: Redis exception; nested exception is io.lettuce.core.RedisException: java.io.IOException: 你的主机中的软件中止了一个已建立的连接。
...
Caused by: java.io.IOException: 你的主机中的软件中止了一个已建立的连接。
	at sun.nio.ch.SocketDispatcher.read0(Native Method)
	at sun.nio.ch.SocketDispatcher.read(SocketDispatcher.java:43)
	at sun.nio.ch.IOUtil.readIntoNativeBuffer(IOUtil.java:223)
	at sun.nio.ch.IOUtil.read(IOUtil.java:192)
	at sun.nio.ch.SocketChannelImpl.read(SocketChannelImpl.java:380)
	at io.netty.buffer.PooledUnsafeDirectByteBuf.setBytes(PooledUnsafeDirectByteBuf.java:288)
	at io.netty.buffer.AbstractByteBuf.writeBytes(AbstractByteBuf.java:1108)
	at io.netty.channel.socket.nio.NioSocketChannel.doReadBytes(NioSocketChannel.java:345)
	at io.netty.channel.nio.AbstractNioByteChannel$NioByteUnsafe.read(AbstractNioByteChannel.java:131)
	at io.netty.channel.nio.NioEventLoop.processSelectedKey(NioEventLoop.java:645)
	at io.netty.channel.nio.NioEventLoop.processSelectedKeysOptimized(NioEventLoop.java:580)
	at io.netty.channel.nio.NioEventLoop.processSelectedKeys(NioEventLoop.java:497)
	at io.netty.channel.nio.NioEventLoop.run(NioEventLoop.java:459)
	at io.netty.util.concurrent.SingleThreadEventExecutor$5.run(SingleThreadEventExecutor.java:886)
	at io.netty.util.concurrent.FastThreadLocalRunnable.run(FastThreadLocalRunnable.java:30)
	at java.lang.Thread.run(Thread.java:748)
```

**解决：** 
- https://blog.csdn.net/Agly_Clarlie/article/details/52251746
- 在启动redis的时候，连同配置文件一起启动`./redis-server redis.conf`

&nbsp;

&nbsp;
