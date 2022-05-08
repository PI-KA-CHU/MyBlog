+++
author = "pikachu"
title = "JSP + Bootstrap +  JAVABean + Servlet + Mysql实现图片上传并分页显示"
date = "2018-11-01"
description = " "
tags = [
    " "
]
categories = [
    "it", "其他"
]

+++

## 一、写在前面

**学习博客：**

- https://blog.csdn.net/qiyuexuelang/article/details/8861300

**需要导入的包：**

- commons-fileupload-1.3.1.jar；
- commons-io-2.4.jar；
- mysql-connector-java-5.1.39-bin.jar；

&nbsp;

## 二、实现流程

### 1. 图片添加界面：add.jsp

```
<%@ page language="java" contentType="text/html; charset=utf-8"
	pageEncoding="utf-8"%>
<!DOCTYPE html>
<jsp:useBean id="DBBean" scope="page" class="com.zeng.lab06.DBBean"/>

<%
	DBBean.getConnect();

	String createSQL = "CREATE TABLE IF NOT EXISTS lab06(id int PRIMARY KEY AUTO_INCREMENT,title varchar(255),author varchar(255),file varchar(255))";
	DBBean.createTable(createSQL);
%>

<html>
<head>
<meta charset="utf-8">
<title>Insert title here</title>
<link rel="stylesheet"
	href="https://cdn.bootcss.com/bootstrap/4.0.0/css/bootstrap.min.css">
<style type="text/css">
#upload {
	width: 30%;
	height: 400px;
	border-width: 1px;
	border-style: inset;
	margin-top: 8%;
	margin-left: 35%;
	background-color: white;
	margin-top: 8%;
}

#form{
	margin-top: 28%;
	margin-left: 10%;
	width: 80%;
}
</style>
</head>

<body>
	<div id="upload">
		<form id="form" action="../UploadServlet" method="post" enctype="multipart/form-data">
			<div class="input-group mb-3">
				<input type="text" class="form-control"
					placeholder="title" aria-label="title"
					aria-describedby="basic-addon2" name="title">
				<div class="input-group-append">
					<span class="input-group-text" id="basic-addon2">标题</span>
				</div>
			</div>
			
			<div class="input-group mb-3">
				<input type="text" class="form-control"
					placeholder="name" aria-label="name"
					aria-describedby="basic-addon2" name="author">
				<div class="input-group-append">
					<span class="input-group-text" id="basic-addon2">发布人</span>
				</div>
			</div>

			<div class="input-group">
				<div class="custom-file">
					<input type="file" class="custom-file-input" id="inputGroupFile04" name="file">
					<label class="custom-file-label" for="inputGroupFile04">choose image</label>
				</div>
				<div class="input-group-append">
					<button class="btn btn-outline-secondary" type="submit">Upload</button>
				</div>
			</div>
		</form>
	
	</div>
	<script src="https://cdn.bootcss.com/jquery/3.2.1/jquery.slim.min.js"></script>
	<script
		src="https://cdn.bootcss.com/popper.js/1.12.9/umd/popper.min.js"></script>
	<script
		src="https://cdn.bootcss.com/bootstrap/4.0.0/js/bootstrap.min.js"></script>
</body>
</html>
```
&nbsp;

### 2. 图片上传的servlet：UploadServlet.java
```
package com.zeng.lab06;

import java.io.File;
import java.io.IOException;
import java.sql.ResultSet;
import java.util.List;

import javax.servlet.ServletContext;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.Cookie;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.apache.commons.fileupload.FileItem;
import org.apache.commons.fileupload.FileUploadException;
import org.apache.commons.fileupload.disk.DiskFileItemFactory;
import org.apache.commons.fileupload.servlet.ServletFileUpload;

@WebServlet("/UploadServlet")
public class UploadServlet extends HttpServlet {
	private static final long serialVersionUID = 1L;

	public UploadServlet() {
		super();
	}

	protected void doGet(HttpServletRequest request, HttpServletResponse response)
			throws ServletException, IOException {

		request.setCharacterEncoding("utf-8");
		response.setContentType("text/html;charset=utf-8");

		// 为解析类提供配置信息
		DiskFileItemFactory factory = new DiskFileItemFactory();

		// 创建解析类的实例
		ServletFileUpload suf = new ServletFileUpload(factory);

		// 设置文件大小
		suf.setFileSizeMax(1024 * 400);

		String title = "";
		String author = "";
		String imgUploadPath = "";
		String fileName = "";
		try {
			List<FileItem> items = suf.parseRequest(request);

			for (FileItem item : items) {

				// isFormField为true，表示这不是文件上传表单域
				if (!item.isFormField()) {

					ServletContext sctx = getServletContext();

					//返回包含给定虚拟路径的实际路径。
					String path = sctx.getRealPath("/upload");

					fileName = item.getName();
					fileName = fileName.substring(fileName.lastIndexOf("\\") + 1);
					imgUploadPath = path + "\\" + fileName;

					System.out.println(imgUploadPath + "=========");
					// "\"会被转义，需要输入"\\\\"用于表示"\"
					imgUploadPath = imgUploadPath.replaceAll("\\\\", "/");

					System.out.println(imgUploadPath + "=========");

					File file = new File(imgUploadPath);
					if (!file.exists()) {
						// 创建文件所需的目录(用于保存上传的图片)
						file.getParentFile().mkdirs();
						item.write(file);

					}
				} else {

					if (item.getFieldName().equals("title")) {
						title = item.getString();
					} else if (item.getFieldName().equals("author")) {
						author = item.getString();
					}
				}

			}

			//保存图片的相对路径
			String rootPath = "/upload/" + fileName;
			String sql = "insert into lab06(title,author,file) " + "values('" + title + "','" + author + "','"
					+ rootPath + "')";
			DBBean.executeUpdate(sql);

			response.sendRedirect("lab06_jsp/list.jsp");

		} catch (FileUploadException e) {
			e.printStackTrace();
		} catch (Exception e) {
			e.printStackTrace();
		}

	}

	protected void doPost(HttpServletRequest request, HttpServletResponse response)
			throws ServletException, IOException {
		doGet(request, response);
	}
}

```

&nbsp;

### 3. 图片分页显示界面：list.jsp
```
<%@page import="java.sql.ResultSet"%>
<%@page import="com.zeng.lab06.DBBean"%>
<%@ page language="java" contentType="text/html; charset=utf-8"
	pageEncoding="utf-8"%>

<%
	String path = request.getContextPath();
	String basePath = request.getScheme() + "://" + request.getServerName() + ":" + request.getServerPort()
			+ path + "/";
%>

<!DOCTYPE html>

<jsp:useBean id="DBBean" scope="page" class="com.zeng.lab06.DBBean" />
<%
	DBBean.getConnect();

	String sqlTotal = "select count(*) from lab06";
	ResultSet rs1 = DBBean.executeQuery(sqlTotal);
	
	int total = 0;
	while(rs1.next()){
		//总的上传数
		total = Integer.parseInt(rs1.getString(1));
	}
	
	
	//起始下标
	int index;
	
	String indexTemp = request.getParameter("index");
	if(indexTemp == null || indexTemp.equals("")){
		index = 0;
	}else{
		index = Integer.parseInt(indexTemp);
	}
	
	//每五个分一页
	int number = 3;
	
%>

<html>
<head>
<meta charset="utf-8">
<link rel="stylesheet"
	href="https://cdn.bootcss.com/bootstrap/4.0.0/css/bootstrap.min.css">
<title>Insert title here</title>

<style>
#main {
	width: 60%;
	margin-top: 5%;
	margin-left: 20%;
	border: 1px inset;
}

#bottom {
	margin-left: 38%;
}

.table {
	text-align: center;
	vertical-align: center;
}
</style>

</head>
<body>
	<div id="main">
		<div id="content">
			<table class="table">
				<thead class="thead-light">
					<tr>
						<th scope="col">#</th>
						<th scope="col">图片</th>
						<th scope="col">标题</th>
						<th scope="col">发布人</th>
						<th>查看详情</th>
					</tr>
				</thead>
				<tbody>
					<%
						if(index < 0){
							index = 0;
						}
					
						String sql = "select * from lab06 limit " + index + "," + number;
						ResultSet rs = DBBean.executeQuery(sql);
						
						String author = null;
						String title = null;
						String img = null;
						while (rs.next()) {
							img = basePath + rs.getString(4);
							title = rs.getString(2);
							author = rs.getString(3);
							
					%>
					<tr>
						<th scope="row"><%=rs.getString(1)%></th>
						<td><img src="<%=img%>"></td>
						<td><%=title%></td>
						<td><%=author%></td>
						<td>
							<a type="button" class="btn btn-success" href="detail.jsp?
							img=<%=img%>&title=<%=title%>&author=<%=author%>">detail</a>
						</td>
					</tr>
					<%
						}
						
						if(index + number > total){
							index = index - number;
						}
						
						//要分的页数
						int pageT = (total % number==0)? (total/number) : (total/number + 1);
					%>
				</tbody>
			</table>
		</div>
		<div id="bottom">
			<nav aria-label="Page navigation example">
				<ul class="pagination">
					<li class="page-item">
						<a class="page-link" href="list.jsp?pageT=<%=pageT%>&total=<%=total%>&index=<%=index-number%>" aria-label="Previous"> 
							<span aria-hidden="true">&laquo;</span>
							<span class="sr-only">Previous</span>
						</a>
					</li>
					<% 
						for(int i = 1;i <= pageT;i ++){
							
					%>
						<li class="page-item"><a class="page-link" href="list.jsp?pageT=<%=i%>&total=<%=total%>&index=<%=number*(i-1)%>"><%=i %></a></li>
					<%
						}
					%>
					<li class="page-item">
						<a class="page-link" href="list.jsp?pageT=<%=pageT%>&total=<%=total%>&index=<%=index+number%>" aria-label="Next"> 
							<span aria-hidden="true">&raquo;</span> 
							<span class="sr-only">Next</span>
						</a>
					</li>
				</ul>
			</nav>
		</div>
	</div>
	<script src="https://cdn.bootcss.com/jquery/3.2.1/jquery.slim.min.js"></script>
	<script
		src="https://cdn.bootcss.com/popper.js/1.12.9/umd/popper.min.js"></script>
	<script
		src="https://cdn.bootcss.com/bootstrap/4.0.0/js/bootstrap.min.js"></script>
</body>
</html>
```

&nbsp;

### 4. 详情图片信息显示页面：detail.jsp
```
<%@ page language="java" contentType="text/html; charset=utf-8"
	pageEncoding="utf-8"%>
	
	
<%
	String path = request.getContextPath();
	String basePath = request.getScheme() + "://" + request.getServerName() + ":" + request.getServerPort()
			+ path + "/";
%>
	
	
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>Insert title here</title>
<link rel="stylesheet"
	href="https://cdn.bootcss.com/bootstrap/4.0.0/css/bootstrap.min.css">

<style type="text/css">
	.card{
		margin-top: 5%;
		margin-left: 30%;
	}
	
	.black{
		margin-left: 85%;
	}
}
</style>
</head>
<body>
	<div>
		<div class="card" style="width: 35rem;">
			<img class="card-img-top" src="<%=request.getParameter("img") %>" alt="Card image cap">
			<div class="card-body">
				<h5 class="card-title">标题：<%=request.getParameter("title") %></h5>
				<p class="card-text">作者：<%=request.getParameter("author") %></p>
				<div class="black"><a href="list.jsp" class="btn btn-primary">返回</a></div>
			</div>
		</div>
	</div>

	<script src="https://cdn.bootcss.com/jquery/3.2.1/jquery.slim.min.js"></script>
	<script
		src="https://cdn.bootcss.com/popper.js/1.12.9/umd/popper.min.js"></script>
	<script
		src="https://cdn.bootcss.com/bootstrap/4.0.0/js/bootstrap.min.js"></script>
</body>
</html>
```