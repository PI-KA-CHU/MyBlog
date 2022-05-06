+++
author = "pikachu"
title = "SVN的forbidden异常"
date = "2018-11-16"
description = "Lorem Ipsum Dolor Si Amet"
tags = [
    "开发工具"
]
categories = [
    "IT",
]
+++


## SVN的forbidden异常

**问题**：将SVN进行relocate时报了forbidden异常，检查后地址等各个都是正确的

**解决**：（首先确认自己的账号密码存在并且正确）缓存导致的问题，需要**清除缓存后checkout导入**：
![svn](https://user-images.githubusercontent.com/38284818/48613799-ad4bf480-e9c7-11e8-97b1-20b66f3bb9d3.JPG)
