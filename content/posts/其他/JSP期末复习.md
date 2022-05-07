+++
author = "pikachu"
title = "JSP期末复习"
date = "2019-01-03"
description = "Lorem Ipsum Dolor Si Amet"
tags = [
    "java"
]
categories = [
    "IT",
]
+++


&nbsp;


## JSP的工作机制

- https://www.jianshu.com/p/93736c3b448b

![1](https://user-images.githubusercontent.com/38284818/50693720-b75aad00-1072-11e9-98fc-9c82bdc58c53.jpg)

![1](https://user-images.githubusercontent.com/38284818/50700283-1a553f80-1085-11e9-929e-3e8e2ca55094.png)

- **基本工作流程：**
浏览器向服务器请求JSP页面，服务器收到该请求后，检查JSP文件内容是不是为第一次访问（是的话则创建servlet），检车JSP文件内容是否被修改（是的话则新建servlet），如果是，则这个JSP文件会在服务器端的**JSP 引擎**作用下转化为一个servlet类的java源代码，然后会在java编译器的作用下转化为一个**字节码文件**，并装载到**JVM**解释运行。

&nbsp;


## JSP的三种脚本元素：

- 声明（declaration）：用于在JSP中声明合法的变量和方法 `<%! 代码内容 %>`
- 表达式（expression）：计算该表达式，将其结果转换成字符串插入到输入中`<%=name %>`
- 脚本（Scriplets）：直接编写一行或多行代码  `<% 代码块 %>`

&nbsp;


## JSP的三种指令元素

- **page指令**：用于声明JSP页面的属性等
```
<%@ page 
 2     [ language="java" ] 
 3     [ extends="package.class" ] 
 4     [ import="{package.class | package.*}, ..." ] 
 5     [ session="true | false" ] 
 6     [ buffer="none | 8kb | sizekb" ] 
 7     [ autoFlush="true | false" ] 
 8     [ isThreadSafe="true | false" ] 
 9     [ info="text" ] 
10     [ errorPage="relative_url" ] 
11     [ isErrorPage="true | false" ] 
12     [ contentType="mimeType [ ;charset=characterSet ]" | "text/html ; charset=ISO-8859-1" ] 
13     [ pageEncoding="characterSet | ISO-8859-1" ] 
14     [ isELIgnored="true | false" ] 
15 %>
```

- **include指令**：通过include指令将包含的源代码添加到页面中
```
静态包含：把其它资源包含到当前页面中。
       <%@ include file="/include/header.jsp" %>
动态包含：
       <jsp:include page="/include/header.jsp"></jsp:include>

原则：能用静态就不要用动态
```

- **taglib指令**：用于引入外部tag标签库后者自定义tag标签库`

```
taglib指令有两个属性，uri为类库的地址，prefix为标签的前缀
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>
```

&nbsp;

## JSP的四个作用域(page，request，session，application)

-  **page：** 当前访问的页面内有效，关闭页面重新打开或刷新后变量或对象重置；

-  **request：** 变量或对象存在于一次完整HTTP请求与响应期间，完成后被释放。当跳转到其他的JSP页面就失效了。

-  **session：** HTTP会话从开始（浏览器打开）到结束这段时间，只要将数据存入session对象，数据的范围就是session

-  **application：** 服务器从启动带停止这一段时间，application作用域上的信息传递是通过ServletContext实现的。

&nbsp;


## JSP的六种常见行为动作

- **<jsp:include >** ：动态包含
- **<jsp:forward >** ：请求转发
- **<jsp:param >** ：设置请求参数

- **<jsp:useBean >** ：创建一个对象
- **<jsp:setProperty >**： 给指定的对象属性赋值
- **<jsp:getProperty >** ：取出指定对象的属性值



&nbsp;


## JSP九大内置对象

![999804-20171008221554356-961080744](https://user-images.githubusercontent.com/38284818/50721528-f4b74d00-10fb-11e9-9030-31f5573a8855.png)

**out：**`out.write("⽂字内容");`

- [ ] 类型：Javax.servlet.jsp.JspWriter
- [ ] 作⽤：主要⽤来向客户端输出数据
- [ ] 作⽤域：page。也就是说，每个⻚⾯都有⼀个⾃⼰的out对象。
- [ ] 重要⽅法：print()/println()/write() 向客户端⻚⾯输出数据

**request：**

- [ ] 类型：Javax.servlet.http.HttpServletRequest
- [ ] 描述：来⾃客户端的请求经Servlet容器处理后，由request对象进⾏封装。注：客户端和服务器的⼀
- [ ] 次通信就是⼀次请求（发送请求或得到相应）。
- [ ] 作⽤域：request。说明，这次请求结束后，它的⽣命周期 就结束了。

```
request.getRequestDispatcher("list.jsp").forward(request,response) 转发(通过代码的⽅式进⾏转发)
request.setCharacterEncoding("UTF-8")  对请求数据重新编码

getParameter(key) 获取提交表单的数据
getParameterValues(key) 获取提交表单的⼀组数据

request.setAttribute(key,object) 设置请求对象的属性
request.gettAttribute(key) 获取请求对象的属性
```


**response：**
- [ ] 类型：Javax.servlet.http. HttpServletResponse
- [ ] 描述：它封闭了JSP 的响应，然后被发送到客户端以响应客户的请求。
- [ ] 作⽤域：page

```
response.sendRedirect("⻚⾯")：⻚⾯跳转。注意，之前的forward是转发，这⾥是跳转，注意区分。
response.setCharacterEncoding("gbk")：设置响应编码
```


**session：**

- **会话的创建过程：**
如果是第⼀次接触“会话”这个概念，需要重复⼀下。说⽩了，客户端与服务器之间可能需要不**断地进行
数据交互**（请求与相应），这个过程就可以理解为⼀段**会话**。Tomcat默认的会话时间为**30分钟**，这段时间内如果没有交互，会话结束；下次客户端⼀旦发送请求，重新创建会话。当客户端第⼀次发送请求的时候，才会创建⼀个会话。**session的⽣命周期⽐request⻓**

- **session创建的详细过程：**
当客户端第一次访问JSP文件的时候，**Servlet容器**（Tomcat）会创建session，如果不是第一次访问，则会检索是否存在此session，存在的话则使用，不存在的话则重新创建，此时为**新会话**。

- [ ] 类型：Javax.servlet.http.HttpSession
- [ ] 描述：表示⼀个会话，⽤来保存⽤户信息，以便跟踪每个⽤户的状态。（不要⽤来保存业务数据，request）
- [ ] 定义：是指在⼀段时间内客户端和服务器之间的⼀连串的相关的交互过程。
- [ ] 作⽤域：session。

```
/*
会话结束的条件之⼀：
服务器关闭
会话过期（⼀段会话时间默认为30分钟）
⼿动终⽌会话
【举例】为保持⽤户登录的状态，我们可以把⽤户的数据信息保存在session中。
*/

session.getid()：取得session的id号.id由tomcat⾃动分配。
session.isnew()：判断session时候是新建的
session.setMaxInactiveInterval(1000*60*30)：设置当前会话失效时间(ms) 。Tomcat默认的会话时间为30分钟。
session.invalidate()：初始化当前会话对象(⼀般在推出的时候使⽤，可以删除当前会话的数据)

session.setAttribute(key,object)：往当前会话中设置⼀个属性
session.getAttribute(key)：获取当前会话中的⼀个属性
session.removeAttribute(key)：删除当前会话中的属性

```


**pageContext：**
- [ ] 类型：javax.servlet.jsp.PageContext
- [ ] 描述：本JSP的⻚⾯上下⽂。
- [ ] 作⽤域：page
- [ ] 注：上下⽂的理解：上下⽂可以联系到当前⻚⾯所有的信息。
- [ ] 我们先来回顾⼀下原始的request.jsp代码：

```
//通过PageContext上下⽂对象获取当前⻚⾯的其他内置对象
pageContext.getRequest();
pageContext.getResponse();
pageContext.getSession();
pageContext.getOut();
String path = request.getContextPath(
```

**application：**

这个对象的⽣命周期是最⻓的。服务器启动的时候就会创建application对象。从服务器存在到服务器
终⽌，都⼀直存在，且只保留⼀个对象，所有⽤户共享⼀个application。不是很常⽤。

- [ ] 类型：javax.servlet.ServletContext
- [ ] 描述：从servlet配置对象获得的servlet上下⽂
- [ ] 作⽤域：application

```
//⼀个应⽤程序只有⼀个application对象
//在服务器启动时创建，到服务器关闭时销毁
//所有客户端共享⼀份
String serverPath = application.getContextPath();//获取当前应⽤程序的路径

//向application对象添加数据
application.setAttribute("", "");

```

**config：**

- [ ] 类型：javax.servlet.ServletConfig
- [ ] 描述：本JSP的 ServletConfig
- [ ] 作⽤域：page
- [ ] 注：代表配置对象，基本⽤不到。

**page：**

- [ ] 类型：java.1ang.Object
- [ ] 描述：实现处理本⻚当前请求的类的实例（javax.servlet.jsp.HttpJspPage），转换后的Servlet类
- [ ] 本身
- [ ] 作⽤域：page

**exception：**

- [ ] JSP常⻅错误状态码：
403：禁⽌访问。⽐如IP地址被拒绝，站点访问被拒绝等
404：找不到。没有找到⽂件或⽬录
500：服务器由于遇到错误⽽不能完成该请求，Web服务器太忙

- [ ] 类型：java.lang.Exception
- [ ] 描述：本JSP⻚⾯的异常对象
- [ ] 作⽤域：page

&nbsp;


## JSP的自定义标签

**自定义标签库方式：**

- **tag方式**，引用的时候通过tagdir属性指定.tag文件的位置，例如
`<%@ taglib prefix="x" tagdir="/WEB-INF/tags" %>`

- **tld方式**，通过uri属性指定.tld文件的位置，需要在web.xml中jsp-config配置taglib
`<%@ taglib prefix="c" tagdir="http://java.sun.com/jsp/jstl/core" %>`

&nbsp;

**自定义标记的方式：**

- 简单标记，不处理体内容：
需实现`javax.servlet.jsp.tagext.Tag`接口，这个接口实现`javax.servlet.jsp.tagext.TagSupport`

- 处理体内容的标记：
需要实现`javax.servlet.jsp.tagext.BodyTag`接口，这个接口实现`java.servlet.jsp.tagext.BodyTagSupport`

**使用步骤：**

- 引入**javax.servlet-api，jsp-api，jstl**三个包
- 新建标签库描述文件，放在**webapp/WEB-INF/tld**目录下
- 标签库描述文件格式：http://book.51cto.com/art/201004/193459.htm

&nbsp;

## 动态include与静态include的区别

- **静态include：** `<%@include file="include.html" %>`
静态include主要是对静态页面的引入，不会检查所包含文件的变化，同时编译时只生成一个class文件（先包含include，后编译）

- **动态include：** `<jsp:include file="include.html" />`
动态include主要是对动态页面的引入，它会检查所引入页面的变化，如果所包含的资源在请求间发生变化，则下次请求此资源时，将包含新的内容。编译后会生成多个class文件。（先编译include的文件，再包含）

![999804-20171008214205871-540005881](https://user-images.githubusercontent.com/38284818/50721154-97b89880-10f5-11e9-92b7-10a4346079f5.png)

&nbsp;

## JSP的常见问题

- **JSP如何把页面的编码格式设为utf-8：** `<%@ page pageEncoding=”utf-8” %>`

- **JSP通常使用什么内置对象实现对用户会话的跟踪：**  `session`

- **taglib用什么方法接收参数，用什么方法发送参数？**
接受参数（EL表达式）：`${param.name}`
发送参数：`<jsp:param name="" value=""/>`

&nbsp;

**参考文献：**

-  https://www.cnblogs.com/zhangyinhua/p/7637399.html#_label1

<hr>
