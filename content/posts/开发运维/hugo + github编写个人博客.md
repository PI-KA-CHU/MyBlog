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
    "it", "开发运维"
]

+++




###  1. hugo + github搭建个人博客

- https://blog.csdn.net/qq_41136216/article/details/112674327

**搭建流程**：

- 下载hugo，并配置环境变量
  - mac下载通过`brew install hugo`命令下载，如果失败，则直接到Hugo github下载extend版本的包
  - 下载完配置环境变量，mac在/etc/paths.d文件夹下添加文件并输入下载的hugo路径
  - `source [新建文件名]`会有权限问题，需要su - root在执行该命令

- `hugo new site myNewSite` 创建新项目
- 在`/content/posts` 文件夹中编写md文件
- 在`/themes` 文件夹中存入主题文件，并复制`/themes/xxx/exampleSite/config.yaml` 文件到`/`下
- 在根目录下执行 `hugo` ，`/public` 文件夹中会生成静态文件，将该文件上传到我们在github中创建的.github.io项目中，即可在打开博客。
- 本地打开：
  - 执行命令根目录下执行 `hugo server -D`
  - 进入：localhost:1313

**配置记录**：

- 头部加入：`draft = true`可以隐藏该md，即不被hugo构建





### 2. github + mac 配置个人密钥



1. 在github的以下位置，生产token

![image](https://raw.githubusercontent.com/PI-KA-CHU/PI-KA-CHU.github.io/master/images/image-20220508095706237.png)



2. mac搜索*Keychain*，进入以下界面，并搜索github

![image](https://raw.githubusercontent.com/PI-KA-CHU/PI-KA-CHU.github.io/master/images/Snipaste_2022-05-08_23-29-40.png)

3. 进入相关的密钥界面，在以下红框内填入第一步生成的密钥并保存即可

![image](https://raw.githubusercontent.com/PI-KA-CHU/PI-KA-CHU.github.io/master/images/image-20220508095706239.png)





### 3. Typora自动上传图片

**问题**：由于博客采用markdown编写，而截图的时候保存图片保存在本地，用hugo生成网站后相对路径发现了变化，这导致图片地址很难控制，即使硬编写后也会导致迁移性很差，在偶然的配置中发现了Typora的内置功能，可以整合PicGo图床上传软件，实现在插入图片时自动上传到图床并生成url。



**解决**：

1. PicGo下载并运行

   - https://github.com/Molunerfinn/PicGo

2. 配置PicGo中的Github图床

   - https://picgo.github.io/PicGo-Doc/zh/guide/config.html#github%E5%9B%BE%E5%BA%8A

3. 在偏好设置中配置PicGo

   ![image-20220509113454353](https://raw.githubusercontent.com/PI-KA-CHU/Image-OSS/main/images/image-20220509113454353.png)

4. 完成后在插入图片/截图时都会自动上传并转化url



**问题**：PICGO上传github图床报127.0.0.1:443异常

**解决**：因为本地用了VPN代理，将PICGO代理设置为本地代理地址后正常。

![image-20220526104543733](https://raw.githubusercontent.com/PI-KA-CHU/Image-OSS/main/images/image-20220526104543733.png)
