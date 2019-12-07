+++
author = "pikachu"
title = "JAVA文件操作相关学习笔记"
date = "2018-10-26"
description = "Lorem Ipsum Dolor Si Amet"
tags = [
    "java"
]
categories = [
    "IT",
]
+++



### File文件转换为MultipartFile 文件：

（可用于测试excel文件导入的接口）

```
File file1 = new File("C:\\Users\\pc\\Desktop\\裁判员信息表.xls");
FileInputStream fis = new FileInputStream(file1);

MultipartFile multipartFile = new MockMultipartFile("file", file1.getName(), "text/plain",IOUtils.toByteArray(fis));
```