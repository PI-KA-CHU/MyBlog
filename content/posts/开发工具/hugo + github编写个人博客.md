+++

author = "pikachu"
title = "hugo + github编写个人博客"
date = "2022-05-06"
description = " "
draft = false
tags = [
	"开发工具"
]
categories = [
    "IT"
]

+++


## 

###  hugo + github编写个人博客

- https://blog.csdn.net/qq_41136216/article/details/112674327

生成流程如下：

- 下载hugo，并配置环境变量
- `hugo new site myNewSite` 创建新项目
- 在`/content/posts` 文件夹中编写md文件
- 在`/themes` 文件夹中存入主题文件，并复制`/themes/xxx/exampleSite/config.yaml` 文件到`/`下
- 在根目录下执行 `hugo` ，`/public` 文件夹中会生成静态文件，将该文件上传到我们在github中创建的.github.io项目中，即可在打开博客。
- 本地打开：
  - 执行命令根目录下执行 `hugo server -D`
  - 进入：localhost:1313



