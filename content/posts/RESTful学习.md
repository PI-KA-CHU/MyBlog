+++
author = "pikachu"
title = "RESTful学习"
date = "2019-02-20"
description = "Lorem Ipsum Dolor Si Amet"
tags = [
	"java"
]
categories = [
    "IT"
]
+++


> **REST**全称是Representational State Transfer，中文意思是**表述性状态转移**，指的是一组`架构约束条件`和`原则`， 如果一个架构`符合REST的约束条件和原则`，我们就称它为RESTful架构。


#### 特征：
- 每个URI代表`一种`资源
- 客户端和服务器之间，传递这种资源的某种`表现层`
- 客户端通过HTTP动词，对服务器端资源进行操作，实现`表现层状态转化`
- URL中通常`不出现动词`，只有`名词`
- 自定义一个占位，可以把摸棱两可的资源请求进行区分,如设计了API为`/{keyId}`和`/{productName}`，正常情况下会出现无法识别的异常，可以修改为`key/{keyId}`和`product/{productName}`


#### 设计方式
- URI的设计技巧
    - 使用`_`或`-`来让URI可读性更好，如：如：`http://www.oschina.net/news/38119/oschina-translate-reward-plan`
    - 使用`/`来表示资源的`层级关系`，如：`/git/git/commit/e3af72cdafab5993d18fae056f87e1d675913d08`
    - 使用`?`用来过滤资源，如：`/pulls?state=closed`用来表示git项目中已经关闭的推入请求
    - `,`或`;`可以用来表示同级资源的关系，如：
`/blocksha1/sha1.h/compare/e3af72cdafab5993d18fae056f87e1d675913d08;bd63e61bdf38e872d5215c07b264dcc16e4febca`


#### 统一资源接口
- 按照HTTP方法的语义来暴露资源，接口将会拥有`安全性`和`幂等性`的特性，例如`GET和HEAD请求都是安全的， 无论请求多少次，都不会改变服务器状态。而GET、HEAD、PUT和DELETE请求都是幂等的，无论对资源操作多少次， 结果总是一样的`，后面的请求并不会产生比第一次更多的影响。
- **HEAD**（获取某个资源的头部信息）

- **GET**（获取资源）
`安全且幂等`
`获取表示`
变更时获取表示（缓存）
200（OK） - 表示已在响应中发出
204（无内容） - 资源有空表示
301（Moved Permanently） - 资源的URI已被更新
303（See Other） - 其他（如，负载均衡）
304（not modified）- 资源未更改（缓存）
400 （bad request）- 指代坏请求（如，参数错误）
404 （not found）- 资源不存在
406 （not acceptable）- 服务端不支持所需表示
500 （internal server error）- 通用错误响应
503 （Service Unavailable）- 服务端当前无法处理请求

- **POST**（创建资源）
`不安全且不幂等`
使用服务端管理的（自动产生）的实例号创建资源
创建子资源
`部分更新资源`
如果没有被修改，则不过更新资源（乐观锁）
200（OK）- 如果现有资源已被更改
201（created）- 如果新资源被创建
202（accepted）- 已接受处理请求但尚未完成（异步处理）
301（Moved Permanently）- 资源的URI被更新
303（See Other）- 其他（如，负载均衡）
400（bad request）- 指代坏请求
404 （not found）- 资源不存在
406 （not acceptable）- 服务端不支持所需表示
409 （conflict）- 通用冲突
412 （Precondition Failed）- 前置条件失败（如执行条件更新时的冲突）
415 （unsupported media type）- 接受到的表示不受支持
500 （internal server error）- 通用错误响应
503 （Service Unavailable）- 服务当前无法处理请求

- **PUT**（更新资源）
`不安全但幂等`
用客户端管理的实例号创建一个资源
通过替换的方式更新资源
如果未被修改，则更新资源（乐观锁）
200 （OK）- 如果已存在资源被更改
201 （created）- 如果新资源被创建
301（Moved Permanently）- 资源的URI已更改
303 （See Other）- 其他（如，负载均衡）
400 （bad request）- 指代坏请求
404 （not found）- 资源不存在
406 （not acceptable）- 服务端不支持所需表示
409 （conflict）- 通用冲突
412 （Precondition Failed）- 前置条件失败（如执行条件更新时的冲突）
415 （unsupported media type）- 接受到的表示不受支持
500 （internal server error）- 通用错误响应
503 （Service Unavailable）- 服务当前无法处理请求

- **DELETE**（删除资源）
`不安全但幂等`
200 （OK）- 资源已被删除
301 （Moved Permanently）- 资源的URI已更改
303 （See Other）- 其他，如负载均衡
400 （bad request）- 指代坏请求
404 （not found）- 资源不存在
409 （conflict）- 通用冲突
500 （internal server error）- 通用错误响应
503 （Service Unavailable）- 服务端当前无法处理请求


#### 使用方式
- GET：http://www.birjemin.com/api/user # 获取列表

- POST：http://www.birjemin.com/api/user # 创建用户

- PUT：http://www.birjemin.com/api/user/{id} # 修改用户信息

- DELETE：http://www.birjemin.com/api/user/{id} # 删除用户信息


#### 过滤信息
- **用于补充规范一些通用字段**

- `?limit=10`：指定返回记录的数量

- `?offset=10`：指定返回记录的开始位置

- `?page=2&per_page=100`：指定第几页，以及每页的记录数

- `?sortby=name&order=asc`：指定返回结果按照哪个属性排序，以及排序顺序

- `?state=close`：指定筛选条件


#### 接口返回示例
- `{"code": 200,"message": "获取成功","succ": true,"data": [] }`


#### 参考文献
- https://baijiahao.baidu.com/s?id=1591221393853082871&wfr=spider&for=pc&isFailFlag=1
- http://www.runoob.com/w3cnote/restful-architecture.html
