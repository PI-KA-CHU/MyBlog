+++
author = "pikachu"
title = "MySQL事务隔离级别及Spring事务管理"
date = "2019-08-04"
description = "Lorem Ipsum Dolor Si Amet"
tags = [
	"java",
	"mysql",
    "spring"
]
categories = [
    "IT"
]
+++



## 一、MySQL事务隔离级别

![image](https://user-images.githubusercontent.com/38284818/62423636-9421da00-b6f5-11e9-8d9e-80a969faca2d.png)


#### 1.1、未提交读（read uncommitted）

![image](https://user-images.githubusercontent.com/38284818/62423550-52446400-b6f4-11e9-8b84-606080dc5367.png)

- **特点**：事务A执行修改语句，但是事务未结束，此时事务B可以读取事务A的修改语句的结果（此时数据库尚未修改）
- **问题**：出现`脏读`现象，如A回滚了，但是B读取到的是A修改的数据。


#### 1.2、已提交读（read committed ）

![image](https://user-images.githubusercontent.com/38284818/62423561-8750b680-b6f4-11e9-9154-4062f530ed0f.png)

- **特点**：事务A执行修改语句且提交事务后，事务B才可以读取。（**SQLServer 、Oracle的默认隔离级别**）
- **问题**：出现`不可重复读问题`，如A执行事务中读取了B事务（已提交）修改的结果，然后在A事务还没结束的时候，B事务再次修改该属性，A事务再次查询时得到了不同的结果。


#### 1.3、可重复读（repeatable read ）

![image](https://user-images.githubusercontent.com/38284818/62423632-80767380-b6f5-11e9-936d-b0815fb494c1.png)

- **特点**：事务A读取事务B的数据，事务B即使在事务A未结束前再次修改，事务A读取的数据仍然是第一次读取到的数据（快照）。（**MySQL默认隔离级别**）
- **问题**：出现`幻读`（插入数据时）问题，如事务A提交了修改某范围内表数据的事务，但是此时事务B插入了某条数据，事务A再次读取时，会出现幻行现象（幻读）。
- **解决**：加上检索范围锁，锁为只可读。


#### 1.4、串行化（serializable）

![image](https://user-images.githubusercontent.com/38284818/62423637-9a17bb00-b6f5-11e9-8dbe-7bdb519a7acb.png)

- **特点**：当启动 `serializable` 隔离级别时，其他事务对表的写操作都将被挂起，故`不会产生不可重复读、幻读、脏读的问题`。但是由于同步化的特性，其`性能较差`，一般不使用。

&nbsp;
&nbsp;

## 二、Spring的事务管理

#### 2.1、Spring事务隔离（5种，如上面的`4种 + 1种默认`，其中的默认隔离是指采用数据库的隔离级别）


#### 2.2、Spring的事务传播行为（7种）

- 事务的传播指`两个事务方法`之间的调用行为

    - **支持当前事务的**：
        - `PROPAGATION_REQUIRED`：存在事务在加入、不存在则开启
        - `PROPAGATION_SUPPORTS`：存在事务则加入
        - `PROPAGATION_MANDATORY`：存在事务则加入、不存在则抛出异常
    - **不支持当前事务的**：
        - `PROPAGATION_REQUIRES_NEW`：存在事务则挂起并开启新事务
        - `PROPAGATION_NOT_SUPPORTED`：存在事务则被挂起
        - `PROPAGATION_NEVER`：存在事务则抛出异常
        - `PROPAGATION_NESTED`：不太明白
&nbsp;

![image](https://user-images.githubusercontent.com/38284818/62423747-0f37c000-b6f7-11e9-8f2a-785e462d0d72.png)

![image](https://user-images.githubusercontent.com/38284818/62423792-01cf0580-b6f8-11e9-958f-502aef4df1bd.png)

&nbsp;

#### 2.3、Spring事务管理（1种编程式事务 + 3中声明式事务）

- **Spring事务管理方法**：
    - **编程式事务**：*一般不用*
        - 1、在XML中配置事务管理相关的Bean，并配置注入模板中
        - 2、在代码中调用注入好的模板的`beginTransaction()`、`commit()`、`rollback()`等事务管理相关的方法
    - **声明式事务**
        - 基于TransactionProxyFactoryBean的方式：*很少用*
        - 基于AspectJ的声明式事务：*常用*
            - **XML方式**：通过XML配置（#63）
            - **注解方式**：通过在业务方法上加`@Transactional`实现

&nbsp;

## 三、参考

- MySQL事务隔离
    - https://baijiahao.baidu.com/s?id=1629344395894429251&wfr=spider&for=pc
    - https://www.jianshu.com/p/4e3edbedb9a8
    - https://blog.csdn.net/qq_35433593/article/details/86094028

- Spring事务管理
    - https://blog.csdn.net/liuwenbiao1203/article/details/52439835
    - 事务传播：https://blog.csdn.net/weixin_39625809/article/details/80707695