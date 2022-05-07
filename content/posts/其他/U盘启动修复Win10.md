+++
author = "pikachu"
title = "U盘修复工具"
date = "2019-12-16"
description = "Lorem Ipsum Dolor Si Amet"
draft = false
tags = [
    "实用工具"
]
categories = [
    "IT"
]
+++

## 问题描述
- 今天一个在暑假的时候认识的朋友加了我微信让我帮忙看下电脑，电脑的情况是：打开一个软件后蓝屏了，并且一直在windows的自动修复下循环，即修复失败，具体错误如下图：
![image.png](http://ww1.sinaimg.cn/mw690/0061iV1igy1g9z0y2irtaj30ku0fm15l.jpg)

&nbsp;

## 解决分析

#### 相似案例查找
- http://tieba.baidu.com/p/5120842696?share=9105&fr=share&unique=BD19F5E3CE7D1EFAE1CBAF2EDA4007C4&st=1576513965&client_type=1&client_version=10.3.16&sfc=copy
- https://www.auslogics.com/en/articles/fix-srttrail-txt-bsod-error-win10/
	- 本人采用修复六

#### 解决方案

1、**关闭Windows自动修复**
- 在修复模式的CMD命令下执行`bcdedit /set {default} recoveryenabled No`关闭自动修复，关闭后重启异常如下：
![image.png](http://ww1.sinaimg.cn/mw690/0061iV1igy1g9z1eedv0dj30la0f0k8z.jpg)

2、**利用U盘启动进行系统引导修复**
- U盘启动盘制作工具（此处用的是老毛桃）： http://www.laomaotao.org/
- 修复流程参考： https://www.fujieace.com/computer-practical/guide.html

3、**利用U盘启动PE删除原系统异常文件**（注意，如果是windows重要文件异常请勿进行）
- 此处主要是要找到系统盘的位置，在步骤2中自动修复的时候会有显示系统盘地址。后根据蓝屏异常文件进行删除。
- 其他流程参考： https://www.fujieace.com/computer-practical/bootsafe-sys.html
- 删除后正常启动

&nbsp;