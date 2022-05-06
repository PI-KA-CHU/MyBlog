+++
author = "pikachu"
title = "Redis命令操作"
date = "2019-01-21"
description = "Lorem Ipsum Dolor Si Amet"
tags = [
	"java",
	"中间件"
]
categories = [
    "IT"
]
+++


## 一、redis的基础命令
- `info`命令：
    - **# Server**：查看系统信息，包括redis的版本信息，使用的端口和process_id和依赖的gcc版本等
    - **# Clients**：客户端连接数量等
    - **# Memory**：内存的使用情况
    - **# Persistence**：持久化备份的参数信息
    - **# State**：redis状态信息
    - **# Replication**：主从同步的相关数据
    - **# Keyspace**：显示已有的空间及空间内的`key`的数量，在redis客户端通过`select index`选择相应的空间，类似于选择数据库表，不同的表中存储着不同的数据
    &nbsp;
    ![image](https://user-images.githubusercontent.com/38284818/51439443-2aac2200-1cf5-11e9-92f0-a42cf3b6a03f.png)
    - 在`redis.conf`文件中可以进行**空间数量**的设置，默认值为**16**：即可以使用的空间是0-15
    &nbsp;
    ![image](https://user-images.githubusercontent.com/38284818/51439386-6abed500-1cf4-11e9-983c-08fa3888b5d2.png)

- `flushdb`命令：清除当前keyspace空间的所有键值
- `flushall`命令：清除所有keyspace空间的所有键值
- `dbsize`命令：当前keyspace的键的数量
- `save`命令：人工触发的对redis的**持久化**
- `quit`命令：退出当前redis客户端的连接

&nbsp;

## 二、redis的键命令
- `set key value`：设置键值
- `del key`：删除键，成功则返回1，失败则返回0
- `exists key`：判断键是否存在，存在则返回1，不存在则返回0
- `ttl key`：time to leave，返回的是键的过期时间，单位时间是秒，若返回-1，则代表该键不会过期，返回-2表示该键不存在
- `expire key time`：设置键的过期时间，如`expire a 10`表示设置键a的过期时间为10秒
- `type key`：返回键的类型，如string，hash等
- `randomkey`：返回随机的key（键）
- `rename key1 key2 `：将键名进行重命名，如`key a b`则将键a的名改为键b，若替换的键名存在，则会将已有的键进行覆盖，即将b的键值覆盖，得到的是`key：b，value：a`
- `renamenx key1 key2`：加了nx结尾的rename命令会进行一定的逻辑判断，该命令会判断该替换的键是否存在，如果存在则返回0，表示替换失败，若不存在则返回1，表示替换成功

&nbsp;

## 三、redis的数据结构

### String字符串
- `setex key time value`：创建键的时候为键设置其有效时间，单位时间为秒，`psetex`设置的时间单位为毫秒
- `getrange key start end`：获取字符串的某一片段（start到end的间字段）
- `getset key value`：为key赋上新的值，返回被替换之前的值。
- `mset key value key value`：进行多个键值的设置
- `mget key1 key2 key3`：进行多个键值的获取，获取到键key1,key2,key3的值
- `strlen key`：返回键值的长度
- `msetnx key value key value`：进行批量的键设置，需要设置的键不存在才会赋值成功，该命令具有**原子性**，及要么全部设置成功返回1，要么都是失败返回0。
- `incr key`：将类型的Integer的键值进行加一操作
- `incrby key number`：为键值一次性加上number的数，如`incr 1 100`，则表示为键为1的值增加100.
- `decr key`：将类型的Integer的键值进行减一操作
- `decrby key number`：为键值一次性减去number的数
- `append key str`：为key的值加上字符串str

&nbsp;

### Hash哈希
- `hset key filed value`：设置哈希键值
- `hexists key field`：判断键是否存在
- `hget key field`：获取键中filed所对应的值
- `hgetall key`：获取键中所有的field和对应的value
- `hkeys key`：获取键中 所有的filed值
- `hvals key`：获取键中所有的value值
- `hlen key`：获取键中filed的数量
- `hmset key field name field name`：批量设置键中的field和value
- `hmget key field field`：批量获取键中field的value
- `hdel key field field`：删除键中的多个field及其值
- `hsetnx key field value`：若存在这个field，则设置失败返回0，否则返回1

&nbsp;

### List列表
- `lpush key value value`：将多个value添加到key中，添加的过程是将value从链表头部一个一个插入，及最后一个加入的是在链表的头部
- `llen key`：获取key的数量
- `lrange key startIndex stopIndex`：获取key列表指定下标范围内的元素
- `lindex index`：获取指定下标的元素值
- `lpop key`：移除key列表的首个（头）元素
- `rpop key`：移除key列表的最后（末尾）元素

&nbsp;

### Set集合
- `sadd key value value`：将多个value加入key集合中
- `scard key`：返回集合中的数量
- `smembers key`：返回集合中的所有成员
- `sdiff key1 key2`：返回key1与key2的差值，即key1减去ke1和key2中共同存在的元素
- `sinter key1 key2`：返回key1与key2的交集
- `sunion key1 key2`：返回key1与key2的并集
- `srandmember key1 number`：返回集合中的随机的number个数
- `sismember key member`：判断member是否为集合key中的元素，是则返回1，不是则返回0
- `srem key value value`：移除集合中的多个元素
- `spop key`：随机移除并返回移除的元素

&nbsp;

### sortedset有序集合
- `zadd key score member score member`：为有序集合添加多个成员元素，score为member的值，且集合按score的值进行排序
- `zcard key`：返回集合中成员的数量
- `zscore key member`：返回member元素的score
- `zcount key min max`：返回score在min到max范围内的member数量
- `zrank key member`：返回member的下标
- `zrange key min max`：返回score在min到max范围内的所有member
- `zrange key min max withscores`：返回score在min到max范围内的所有member及其对应的score


