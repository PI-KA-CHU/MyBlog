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

## 

## SpringBoot整合WebSocket

- https://blog.csdn.net/moshowgame/article/details/80275084



整合websocket可能遇到的问题：

- 功能上：可行
  - 什么时候主动推送事件？如果是有事件进来的时候推送，是触发的时候去数据库（存在事件还没有插入数据库的问题）？如果不是查数据库而是直接推给前端，则只能单独发送一条，所以未读信息需要前端维护。
  - 如何经过网关建立websocket？
  - 集群环境下如何共享websocket session？目前猜测，将session存放在redis中，集群从redis中获取。