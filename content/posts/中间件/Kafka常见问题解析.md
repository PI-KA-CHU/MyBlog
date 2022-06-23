+++
author = "pikachu"
title = "Kafka常见问题解析"
date = "2022-06-23"
description = " Kafka顺序消费、幂等性、消息可靠性"
tags = [
	"kafka"
]
categories = [
    "it","中间件"
]

+++



## 一、顺序消费



### Kafka是如何消费消息的

- Kafka消费线程模型：https://juejin.cn/post/6906843890607718407



**Kafka的topic、partition、consumer的关系**

- topic：partition - 1：n【Kafka中一个topic对应多个partition】

- partition：consumer - n：1【一个partition只能被一个consumer消费，一个consumer可以消费多个partition，consumer采用的是单线程拉取消息，每次默认拉取500条到本地，保证消息的有序性，但是单线程导致消息的吞吐量较低】



### 如何保证消息有序

- 如何确保消息有序性：https://juejin.cn/post/6962323755959844894

消息有序由以下两方面控制：

- 生产者发送的消息有序
- Kafka分配到partition的消息有序：**通过partitionKey进行分配**
- 消费者消费有序：Kafka的partition由同个consumer消费，并且consumer默认采用单线程进行消息拉取和消息，所以本质上是有序的





### 如何保证有序的同时提高消息吞吐

Kafka consumer默认采用单线程进行消息处理，如果对消息吞吐有更高的要求，可以在接收到消息后再根据业务需要再次进行哈希分组，将同个partition中的消息进行分组，并采用多线程分别对每个分组进行消息，这样可以提高消息的消费速度。







