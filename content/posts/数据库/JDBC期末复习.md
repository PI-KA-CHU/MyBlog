+++
author = "pikachu"
title = "JDBC期末复习"
date = "2019-01-05"
description = " "
tags = [
    "mysql"
]
categories = [
    "it","数据库"
]

+++


&nbsp;

## JDBC操作数据库的一般步骤

- 加载正确的数据驱动程序
`Class.forName("com.mysql.jdbc.Driver")`
- 定义所要连接数据库的地址
`String url = "jdbc:mysql://localhost:3306/dbname?SSL=false"`
- 建立与数据库的连接
`Connection conn = DriverManager.getConnection(url,username,password);`
- 创建语句对象
`Statement stmt = conn.createStatement();`
- 声明SQL语句，并将该语句通过Statement对象提交给服务器进行执行
```
String sql = "SELECT name FROM t_student";
ResultSet rs= stmt.executeQuery(sql);
```
- 对查询结果进行分析
```
while(rs.next()){
name = rs.getString("name");
phone = rs.getString("sex");
}
```
- 关闭打开的资源
```
rs.close();
stmt.close();
conn.close();
```

&nbsp;
&nbsp;


## Statement的三种类型
**Statement：常规语句**

- 普通的不带参的查询SQL

- 支持批量更新和删除

**PreparedStatement：预置语句**

- 可变参数的SQL，编译一次，执行多次，效率高

- 安全性好，有效防止SQL注入等问题

- 支持批量更新和删除

```
PreparedStatement stmt = connection.prepareStatement("insert into test values('?,?')");
stmt.setString(1,”first value”);
stmt.setString(2,”second value”);
stmt.executeUpdate();
```

**CallableStatement：可调用语句**

- 继承自PreparedStatement，支持带参数的SQL操作；

- 支持调用**服务器上的存储过程**，提供了对输入/输出参数（IN OUT）的支持

```
// 创建有输入输出参数的存储过程：
create procedure getNameById(in cid int,out return_name varchar(20)) select name from usertbl where id = cid;
CallableStatement cstmt = conn.prepareCall("{call getNameById(?,?)}");  // 输入输出变量都要以?表示
cstmt.setInt(1, 20);   // 给第一个问号赋值为20

// 注册输出参数
cstmt.registerOutParameter(2, Types.CHAR);   // 给作为输出变量的第二个问号注册，并制定其为char型
cstmt.execute();   // 执行 call getNameById()
String name = cstmt.getString(2);   // 取存储过程中的第二个变量的值
```


&nbsp;
&nbsp;


## 网络服务可以采用技术来提高系统性能

- **负载均衡：** 如nginx
- **缓存技术：** 如redis
- **连接池技术：** 如数据库连接池c3p0，hikari
- **容错机制**


&nbsp;
&nbsp;


## 数据库连接池的工作机制

- 数据库连接池在初始化时将**创建一定数量的数据库连接**放到**连接池**中，这些数据库连接的数量是有**最小数据库连接数**决定的。

- 无论这些数据库连接是否被使用，连接池都将**一直保证至少拥有这么多的连接数量**。

- 连接池的**最大数据库连接数量**限定了这个连接池能占有的**最大连接数**。

- 当应用程序向连接池请求的连接数超过最大连接数量时，这些请求将被加入到**等待队列**中。


&nbsp;
&nbsp;


## 数据库中事务理解：（JAVA中对事务的处理）

- **原子性：** 原子性是指事务中的操作要么全做，要么都不做，保证数据库的一致性

- **一致性：** 一致性是指数据库在事务在操作前后的数据都必须满足业务规则约束。

- **隔离性：** 隔离性是数据库允许多个并发食物同时对齐数据进行读写和修改的能力，隔离性防止了多个事务并发执行时由于交叉执行而导致的数据库不一致。

- **持久性**：持久性表示为：事务处理结束后，对数据库的修改就是永久的，即便系统故障也不会丢失。
