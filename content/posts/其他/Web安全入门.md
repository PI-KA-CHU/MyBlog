+++
author = "pikachu"
title = "Web安全入门"
date = "2019-12-07"
description = "Lorem Ipsum Dolor Si Amet"
draft = true
tags = [
	"安全"
]
categories = [
    "IT"
]
+++

## 一、Web安全入门

#### Web安全案例

![1.JPG](http://ww1.sinaimg.cn/mw690/006H3ec5gy1g9o6vaa8zbj30hz0dlaao.jpg)
> 1、某诈骗集团通过短信发送了钓鱼网站给我们，以下是通过一系列的手段最终追踪到用户的手段

- User-Agent：用电脑模拟手机User-Agent登陆网站
- XSS获取管理后台权限
  - XSS为跨站脚本攻击，获取其后台管理凭证，比如获取到其后台管理地址及后台管理凭证（cookie）
- 利用JSP漏洞，获取其电脑登陆过的其他网站的账号，比如获取到保存在cookie中的百度username
- 通过获取的百度贴吧username人肉它，搜索其发过的贴子，进而获取其个人信息，甚至是相关的联系方式等。

#### 安全证书
- https://zhuanlan.zhihu.com/p/21750494
  - CISP：中国
  - CISSP：国际化

#### 常见的攻击手段
![2.JPG](http://ww1.sinaimg.cn/mw690/006H3ec5gy1g9o6yt6rg8j30pd0dsdgu.jpg)

#### 企业攻防对抗示例

![3.JPG](http://ww1.sinaimg.cn/mw690/006H3ec5ly1g9o2qvzi4nj30op0e9dh4.jpg)

&nbsp;

## 二、Web安全实战

#### 1、寻找邮箱信息
> 给你一个邮箱地址，如何找到和这个邮箱相关的更多信息

- 通过百度或者谷歌搜索邮箱账号，获取相关信息
- 通过Firefox Monitor查看个人数据是否泄漏
	- https://monitor.firefox.com/

#### 2、查找网站后台

- 准备
	- 公司的网站是？
	- 后台的关键字
	- 利用`搜索引擎、社工库、社交论坛等`查找
- 搜索引擎
	- 类别
		- baidu.com（过滤比较严重）
		- google.com（语法强大，挖掘能力较好）
		- duckduckgo.com（推荐，挖坟能力简直不能强大，把祖坟都挖出来了）
	- 命令
		- `site:edu.cn`：包含`edu.cn`域名的网站
		- `inurl:admin`：网页中包含admin的网站
		- `intitle:奖励`：搜索网站标题中包含奖励的网页
		- `filetype:pdf`：只搜索指定类型的文档的网页
		- 其他谷歌命令参考： https://www.zhihu.com/question/20161362


## 金句
- 始于兴趣，成于坚持

