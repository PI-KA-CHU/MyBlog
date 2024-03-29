+++
author = "pikachu"
title = "Flume收集日志 - windows（实战版一）"
date = "2019-05-06"
description = " "
draft = false
tags = [
    "大数据"
]
categories = [
    "it"
]

+++



**flume官方文档**：
- http://flume.apache.org/releases/content/1.9.0/FlumeUserGuide.html

**技术博客（较全面的介绍）**：
- http://www.51niux.com/?id=197

&nbsp;

## 一、Flume的核心组件

- **source**：用于对源文件的监控（即数据的传入点）
- **channel**：用于event数据的传输（即传输管道）
- **sink**：用于event数据的输出（即输出点）

<br>

### 1.1 Flume的source类型

- https://www.cnblogs.com/swordfall/p/8254271.html
- https://www.cnblogs.com/qingyunzong/p/8995554.html
- http://flume.apache.org/FlumeUserGuide.html#flume-sources（**官方文档**）

    source类型的种类比较多，上面是关于各种类型的介绍，比较常用的是avor、thrift、exec、jms、spool，本次记录使用的是spool，spool是对**本地文件的监视**，即监视文件夹中的文件，将新加入的文件进行处理上传，成功处理后的文件默认后缀会加上**.COMPLETED**（可自定义修改，也可修改为在上传后删除）。

<br>

### 1.2 Flume的chancel类型

- **Memory Channel**：基于内存传输，实现高速吞吐，但无法保证数据的完整性。
- **JDBC Channel**：事件会被**持久化**（存储）到可靠的数据库里，目前支持嵌入式Derby数据库，但是该数据库使用起来不太方便，目前不适用于生产环境。
- **File Channel**：基于磁盘传输（**持久化**隧道），将事件存储于磁盘中，即使宕机也能保证数据的完整性，即数据不会丢失。
- **Psuedo Transaction Channel**（不常见）
- 相关参数及详情参考最上方的官方文档及技术博客

<br>

### 1.3 Flume的sink类型

- https://www.cnblogs.com/swordfall/p/8157766.html

    sink类型同样有很多种，如hdfs sink、hbase sink、avor sink、logger sink等（详情参考上面链接），本案例使用的是hdfs sink，即将日志数据通过flume收集传输到hdfs。
    

&nbsp;

## 二、Flume的常见使用

<br>

### 2.1 多层代理

![image](https://user-images.githubusercontent.com/38284818/57234496-56a8a480-7053-11e9-8d6e-223dff119718.png)

<br>

### 2.2 并流

- 从大量Web服务器收集的日志发送给写入HDFS集群的十几个代理，即收集服务器集群的日志文件，汇流到代理flume并存入hdfs

![image](https://user-images.githubusercontent.com/38284818/57234574-8788d980-7053-11e9-93e3-9b1b96be2638.png)

<br>

### 2.3 多路复用

- 将事件流**多路复用**到一个或多个目的地。这是通过定义可以复制或选择性地将事件路由到一个或多个信道的流复用器来实现的。即同一个数据源的日志文件我们可能需要分配到多处进行处理，或者说有多个子系统需要使用到，此时可以使用到多路复用，sink指向不同的目的地。

![image](https://user-images.githubusercontent.com/38284818/57234813-1138a700-7054-11e9-8cf9-1dd6acfa3213.png)


&nbsp;


## 三、flume的相关配置

### 3.1 flume-env.sh文件

- 如果Hadoop相关支持包已经导入到flume的lib中，则配置flume的lib路径
```
$FLUME_CLASSPATH="D:\flume\apache-flume-1.8.0-bin\lib"
```

- 如果Flume/lib中没有Hadoop相关支持包，则需要指定本地Hadoop路径
```
HADOOP_HOME="G:\\hadoop-2.5.0-cdh5.3.6"
FLUME_CLASSPATH="$HADOOP_HOME/share/hadoop/hdfs//hadoop-hdfs-2.5.0-cdh5.3.1.jar"
```

<br>

### 3.2 example.conf文件

```

# flume相关组件的声明
a1.sources = r1
a1.sinks = k1 k2
a1.channels = c1 c2

# 设置source r1
# fileHeader：是否添加绝对路径的标头
# inputCharset：传入event的编码格式
# ignorePattern：忽略的文件名，此处匹配的文件不会被传输（可用正则）
# includePattern：包含的文件名，此处匹配的文件会被传输（可用正则）
a1.sources.r1.type = spooldir
a1.sources.r1.spoolDir = E:\\BigDataSolf\\log\\uploadTest
a1.sources.r1.fileHeader = true
a1.sources.r1.inputCharset  = GBK
# a1.sources.r1.ignorePattern = catalina.*
a1.sources.r1.includePattern = app.*


# 配置r1拦截器(自定义拦截器)，自定义拦截器需要打成jar包后放在flume的lib文件夹中，后面会有自定义拦截器的介绍。
# type：拦截器类型（此处为自定义的拦截器）
# regexs和datePattern：都是自定义的正则参数
a1.sources.r1.interceptors = i1
a1.sources.r1.interceptors.i1.type = com.bnuz.flume.MyLogBuilder
a1.sources.r1.interceptors.i1.regexs = localhost_access_log,app
a1.sources.r1.interceptors.i1.datePattern = [0-9]{4}-[0-9]{2}-[0-9]{2}

# 配置多路复用选择器（根据头信息进行分流）
# header：指header中的参数名称（类似于key）
# mapping.localhost_access_log：localhost_access_log相当于value，即如果myHeader匹配的value为localhost_access_log就使用c1管道，如果是app则使用c2管道。
a1.sources.r1.selector.type = multiplexing
a1.sources.r1.selector.header = myHeader
a1.sources.r1.selector.mapping.localhost_access_log = c1
a1.sources.r1.selector.mapping.app = c2


# Use a channel which buffers events in memory for c1
a1.channels.c1.type = memory
a1.channels.c1.capacity = 8000
a1.channels.c1.transactionCapacity = 1000

# Use a channel which buffers events in memory for c2
a1.channels.c2.type = memory
a1.channels.c2.capacity = 8000
a1.channels.c2.transactionCapacity = 1000


# configure the sink k1
# %{fileDate}是自定义拦截器中加入的日期参数，用于将日志文件按照日期保存到不同的文件夹中
# 将access_log文件传输到access_log文件夹中
# fileSuffix：保存的文件前缀
# fileSuffix：保存的文件后缀
# useLocalTimeStamp：为每个文件名加上存储时的时间戳
a1.sinks.k1.type=hdfs
a1.sinks.k1.hdfs.path=hdfs://127.0.0.1/logs/zhsj/%{fileDate}/access_log
a1.sinks.k1.hdfs.fileType=DataStream
a1.sinks.k1.hdfs.writeFormat=TEXT
a1.sinks.k1.hdfs.filePrefix=%{fileDate}
a1.sinks.k1.hdfs.fileSuffix=.log
a1.sinks.k1.hdfs.rollInterval=0
a1.sinks.k1.hdfs.rollSize=10240000
a1.sinks.k1.hdfs.rollCount=0
a1.sinks.k1.hdfs.minBlockReplicas=1
a1.sinks.k1.hdfs.useLocalTimeStamp = true

# configure the sink k2
a1.sinks.k2.type=hdfs
a1.sinks.k2.hdfs.path=hdfs://127.0.0.1/logs/zhsj/%{fileDate}/app_log/
a1.sinks.k2.hdfs.fileType=DataStream
a1.sinks.k2.hdfs.writeFormat=TEXT
a1.sinks.k2.hdfs.filePrefix=%{fileDate}
a1.sinks.k2.hdfs.fileSuffix=.log
a1.sinks.k2.hdfs.rollInterval=0
a1.sinks.k2.hdfs.rollSize=10240000
a1.sinks.k2.hdfs.rollCount=0
a1.sinks.k2.hdfs.minBlockReplicas=1
a1.sinks.k2.hdfs.useLocalTimeStamp = true


# Bind the source r1 and sink to the channel
# 绑定不同的channel、sink和source间的关系
a1.sources.r1.channels = c1 c2
a1.sinks.k1.channel = c1
a1.sinks.k2.channel = c2

```

&nbsp;

## 四、flume的自定义拦截器

- 此拦截器用于对event的简单过滤，并为不同文件和时间的event进行标识，flume配置文件（如上）会根据参数的不同保存到不同的文件夹中。将写好的拦截器打包成`jar包`并保存到flume的`lib`文件夹中。

<br>

MyLogBuilder（拦截器入口）

```
import org.apache.flume.Context;
import org.apache.flume.interceptor.Interceptor;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class MyLogBuilder implements Interceptor.Builder{
    private Logger log = LoggerFactory.getLogger(MyLogBuilder.class);
    private String regexs = "";
    private String datePattern = "";

    //获取传入的相关参数
    @Override
    public void configure(Context context) {
        regexs = context.getString("regexs");
        datePattern = context.getString("datePattern");
        log.info("------获取到拦截器参数pattern为：" + regexs);
    }
    
    //调用拦截器并传入参数
    @Override
    public Interceptor build() {
        log.info("------初始化自定义拦截器");
        return new MyLogInterceptor(regexs,datePattern);
    }
}
```
MyLogInterceptor（拦截器的实现）
```
import org.apache.commons.io.Charsets;
import org.apache.flume.Event;
import org.apache.flume.interceptor.Interceptor;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.*;
import java.util.regex.Matcher;
import java.util.regex.Pattern;

public class MyLogInterceptor implements Interceptor {

    private Logger log = LoggerFactory.getLogger(MyLogInterceptor.class);

    //文件地址（包含文件名）
    private String filePath = "";

    //文件类型过滤参数
    private String regexs = "";

    //日期格式（正则表达式）
    private String datePattern = "";

    //日志日期键值
    private final String FILE_DATE_KEY = "fileDate";
    private String fileDateValue = "";

    //日志类型键值
    private  final String HEADER_KEY = "myHeader";
    private String headerValue = "";

    MyLogInterceptor(String regexs,String datePattern){
        this.regexs = regexs;
        this.datePattern = datePattern;
    }

    @Override
    public void initialize() {
        log.info(regexs + "拦截器initialize方法执行");
    }

    /**
     * 对每个事件进行拦截修改
     *
     * @param event 文件事件
     * @return
     */
    @Override
    public Event intercept(Event event) {


        /**
         * 根据不同的日志文件内容进行不同的正则表达式匹配
         * access日志文件每天生成一次，则直接使用文件名进行日期匹配（文件名包含日志）
         * app日志文件几天生成一次文件，根据具体日志数据进行匹配
         */
        Pattern pattern = pattern = Pattern.compile(datePattern);
        Matcher matcher = null;
        if (filePath.contains("localhost_access_log")){
            //通过正则表达式获取日志创建日期
            matcher  = pattern.matcher(filePath);
        }else if(filePath.contains("app.")){
            //日志具体数据
            String bodyData = new String(event.getBody(), Charsets.UTF_8);
            log.info("收集到的数据为：{}",bodyData);
            matcher = pattern.matcher(bodyData);
        }

        if (matcher != null && matcher.find()){
            log.info("匹配到日期为：" + matcher.group(0));
            fileDateValue = matcher.group(0);

            event.getHeaders().put(HEADER_KEY,headerValue);
            event.getHeaders().put(FILE_DATE_KEY,fileDateValue);

            return event;
        }else {
            log.info("信息被过滤");
            return null;
        }
    }

    @Override
    public List<Event> intercept(List<Event> list) {

        List<Event> result = new ArrayList<>();
        //获取文件名
        filePath = list.get(0).getHeaders().get("file");
        
        //将传入的过滤参数进行切割
        String[] regexArr = regexs.split(",");
        for (String s : regexArr) {
            if (filePath.contains(s)) {
                log.info("合法文件，允许放行 - {}", filePath);

                //设置header键值，用于多路复用（source根据头键值不同进行不同channel的选择）
                headerValue = s;
                Event event1;
                for (Event event : list){
                    event1 = intercept(event);
                    if (event1 != null){
                        result.add(event1);
                    }
                }
                return result;
            }
        }
        log.info("拦截器拦截非法文件 - {}",filePath);
        return result;
    }

    @Override
    public void close() {
        log.info("拦截器关闭");
    }

}
```


&nbsp;

## 五、windows下flume的启动

- 打开cmd命令进入flume的bin文件中

- 使用如下命令启动：（windows启动需要注释掉部分代码）
```
flume-ng agent --conf ../conf --conf-file ../conf/example.conf --name a1 -property flume.root.logger=INFO,console
```

&nbsp;

## 六、异常及解决：

- 参考：`Flume收集日志 - windows（配置版）`



