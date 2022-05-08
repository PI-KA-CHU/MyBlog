+++
author = "pikachu"
title = "Cookie,Session及Application的使用"
date = "2018-10-02"
description = " "
tags = [
    ""
]
categories = [
    "it", "spring"

]

+++


&nbsp;

## 1.使用Cookie记录页面浏览次数（通过按钮刷新的方式）
&nbsp;

```
<%@ page language="java" contentType="text/html; charset=utf-8"
    pageEncoding="utf-8"%>
<%
	
	/**
		功能：使用cookie记录页面的浏览次数：
		步骤：
		1.遍历所有cookie：
		2.count用于记录浏览的次数，如果存在就为此次访问count + 1， 没有的话count默认为1，表示第一次访问。
		3.计算完count值后创建cookie并把count的值赋值进去。
		（cookie名相同的会自动替代原有的，不过各个属性设置要相同）
		
	**/
	int count = 1;
	for(int i = 0;i < request.getCookies().length;i ++){ 
		Cookie cookie = request.getCookies()[i];
		if(cookie.getName().equals("count")){
			count = Integer.parseInt(cookie.getValue());
			count ++;
		}
	}
	Cookie visitCookie = new Cookie("count",Integer.toString(count));
	response.addCookie(visitCookie);
%>
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>使用cookie记录每次浏览</title>
</head>
<body>
	<div>
		<label>阅览次数：</label><%= count %><br/>
		<!-- 如果用response.send()去跳转，需要将cookie设置为setPath("/")，表示cookie适用于整个项目 -->
		<input type="button" onclick="location='<%= request.getRequestURI() %>'" value="刷新">
	</div>
</body>
</html>
```

附：
- http://www.51gjie.com/javaweb/860.html
- https://segmentfault.com/a/1190000004556040

&nbsp;&nbsp;
&nbsp;
&nbsp;&nbsp;

## 2.使用session统计现在访问的人数

#### a.创建一个含有静态（static）值的类：计数

    `public static int onlinePeople = 0;`

#### b.创建一个实现HttpSessionAttributeListener接口的类：监听</h4>

```
public class OnlineCountListener implements HttpSessionAttributeListener {

	//用于监测当有session的值被添加的使用
	@Override
	public void attributeAdded(HttpSessionBindingEvent event) {
		System.out.println("正常监测到" + event.getName());
		if(event.getName().equals("userIP")) {
			System.out.println("session值有添加IP");
			OnlineCountInfo.addOnlinePeople();
		}
	}

	//用于监测session的值被删除（或者过期）的时候
	@Override
	public void attributeRemoved(HttpSessionBindingEvent event) {
		if(event.getName().equals("userIP")) {
			System.out.println("session值有删除IP");
			OnlineCountInfo.reduceOnlinePeople();
		}
	}

	//用于监测session值被替换的时候
	@Override
	public void attributeReplaced(HttpSessionBindingEvent event) {
		
	}

}
```
#### c.将创建的监听的类注册到web.xml里面：注册

```
<listener>
    	<listener-class>com.bnuz.test.util.OnlineCountListener</listener-class>
</listener>
```

#### d.在JSP中使用session进行统计在线人数：统计

步骤：
-  在login.jsp使用loginServlet进行模拟登陆
-  在servlet中进行操作
-  跳转到successLogin.jsp页面


LoginServlet.java：
```
protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
	String username = request.getParameter("username");
	String password = request.getParameter("password");
	
	System.out.println(getIpAddr(request));
	
	//模拟登陆
	if(username.equals("") || username.trim().equals("")) {
		response.sendRedirect("login.jsp");
	}else {
		
		HttpSession session = request.getSession();
		//设置session的有效时间为10秒（刷新则会重新计数），超过10秒无交互视为下线
		session.setMaxInactiveInterval(10);
		
		if(session.getAttribute("userIP") == null) {
			//跳过代理，获取客户端IP
			String IP = getIpAddr(request);
			session.setAttribute("userIP", IP);
		}
		
		response.sendRedirect("successLogin.jsp");
	}
	
}


//来源：http://blog.51cto.com/drizzlewalk/479934
//越过多级代理,获取客户端真实IP
public String getIpAddr(HttpServletRequest request) {

    String ip = request.getHeader("x-forwarded-for");
    if(ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
        ip = request.getHeader("Proxy-Client-IP");
    }
    if(ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
        ip = request.getHeader("WL-Proxy-Client-IP");
    }
    if(ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
        ip = request.getRemoteAddr();
    }
    return ip;

}
```

successLogin.jsp：

```
<body>
	<%
		
		//每11秒刷新一次
		response.setIntHeader("Refresh", 11);
		Date date = new Date();
		DateFormat format = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
		
		HttpSession session1 = request.getSession();
		
		//在线人数
		String times = "登陆超时，请重新登陆";
		try{
			if(session1.getAttribute("userIP") != null){
				times = OnlineCountInfo.getOnlinePeople() + "";
			}
		}catch(NullPointerException e){
			System.out.println("没有值");
		}
	%>
	<h2><%=format.format(date) %></h2>
	<h2>当前在线人数：<%=times %></h2>&nbsp;
	<h2>登陆成功</h2>
</body>
```


## 3.使用Application存储不同账号（浏览器）登陆对象
&nbsp;

Application存在于整个服务器阶段，除非重开服务器（Tomcat），否则不会刷新

```
//获取application
ServletContext application = this.getServletContext();

//为application赋值
application.setAttribute(name, value);

//移除application
application.removeAttribute(name);
```