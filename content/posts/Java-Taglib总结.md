+++
author = "pikachu"
title = "Java-Taglib总结"
date = "2018-12-28"
description = "Lorem Ipsum Dolor Si Amet"
tags = [
    "java"
]
categories = [
    "IT",
]
+++



### JSTL依赖包下载（本人使用1.1.2版本）

- https://archive.apache.org/dist/jakarta/taglibs/standard/binaries/

### 使用流程：

- 将依赖包中lib文件夹下的**jstl**和**standard**的jar包导入项目

- 在JSP头部引入taglib相关的地址：

```
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>  //基本taglib
<%@ taglib prefix="sql" uri="http://java.sun.com/jsp/jstl/sql"%>  //数据库操作相关taglib
<%@ taglib prefix="fn" uri="http://java.sun.com/jsp/jstl/functions" %> //相关函数taglib
```

### 基本taglib的用法总结
<!-- 连接数据库 -->

```

- taglib操作数据库，使用${ }可以引用taglib定义的标签的变量名：

<sql:setDataSource var="myDB" driver="com.mysql.jdbc.Driver" url="jdbc:mysql://localhost:3306/test" user="root" password="19970121" />

<sql:query dataSource="${myDB}" var="totalList">SQL语句</sql:query>

<sql:query dataSource="${myDB}" var="list">${sql}</sql:query>


/**************************************************************************************************/


- 使用taglib基本操作：

/* 设置某一变量的值 */
<c:set var="sql" value="SELECT id FROM t_student WHERE username='曾'"></c:set>

/* IF语句的使用（只有条件成立才会执行里面的程序）*/
<c:if test="${authorName ne null and !empty authorName}">
    ${sql.concat("AND a.author LIKE '").concat(authorName).concat("%'") }
</c:if>

/* forEach的使用（list为执行sql语句后的var）*/
<c:forEach var="item" items="${list.rows}" begin="0" end="10">		
    <tr>
	<th scope="row">${item.id}</th>
	<td><img src="${item.imageUrl}" class="td-img"></td>
	<td>${item.title}</td>
	<td>${item.author}</td>
    </tr>
</c:forEach>

/* choose的使用（类似于switch）*/
<c:choose>
    <c:when test="${(total % number) eq 1}">
	<c:set var="pageT" value="${total/number}"></c:set>
    </c:when>
   <c:when test="${(total % number) eq 2}">
	<c:set var="pageT" value="${total/number}"></c:set>
    </c:when>
    <c:otherwise>
	<c:set var="pageT" value="${total/number + 1}"></c:set>
    </c:otherwise>
</c:choose>

```

