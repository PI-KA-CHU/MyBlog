+++
author = "pikachu"
title = "Idea的使用及常见问题"
date = "2019-01-18"
description = " "
tags = [
	"", "开发工具"
]
categories = [
    "it", "开发运维"
]

+++


## 一、前言
> 一直使用Eclipse进行java开发的我最近发现一个界面及其炫酷的编译器Idea，尝试着使用Idea进行开发，记录下出现的一些问题


## 二、Idea导入Eclipse项目（SpringBoot）


**1. bug：**
- 开始导入的时候是直接在打开项目文件（错误的打开方式），然后一步一步进行配置
，出现了`No mapping found for HTTP request with URI [/] in DispatcherServlet 
with name 'dispatcherServlet'`的问题。

**2. 解决：**
- 百度谷歌查了各种办法，捣鼓了大半天都没解决，最后把整个项目**删除了重新导入**，正常了！
问题出在导入方式有误，导致配置文件没有正常配置。

- 首先在主界面导入项目`Import Project`，进入后选择 `Eclipse`（如果是Eclipse项目的话），导入后运行即可正常访问。

![2](https://user-images.githubusercontent.com/38284818/51367094-fe8d7700-1b23-11e9-971b-dde1c7cae574.JPG)

&nbsp;
&nbsp;

## 三、Idea实现热部署（4种）
- 参考：https://www.cnblogs.com/jcook/p/6910238.html
1. **修改服务器配置**，使得IDEA窗口失去焦点时，更新类和资源（只在`debug`启动模式下有效）

    - 点击Idea上方的Run菜单栏，点击Edit Configuration
    - 在`Deployment`栏目中点击右边的 '+' 号选择相应的war包
    - 将`On Updatae action`和`On frame deactivation`设置为`Update classes and resources`
    &nbsp;
    ![image](https://user-images.githubusercontent.com/38284818/51897059-ccfc9180-23e8-11e9-860a-e51fd7468f48.png)

&nbsp;

2. **使用springloaded jar包**

    - 下载jar包并导入项目：https://github.com/spring-projects/spring-loaded
    -  启动应用时添加VM启动参数：`-javaagent:/home/lkqm/.m2/repository/org/springframework/springloaded/1.2.7.RELEASE/springloaded-1.2.7.RELEASE.jar -noverify`

&nbsp;

3. **使用spring-boot-devtools提供的开发者工具**

    - 在springBoot项目中加入如下依赖：
    ```
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
    </dependency>
    ```
    - 如果是Idea编译器，则需要启动Idea的**自动编译**，否则不会生效：https://blog.csdn.net/u012190514/article/details/79951258

&nbsp;

4. **使用Jrebel插件实现热部署**(该插件14天免费试用)

    - 点击Idea左上角`File -> Setting -> Plugin`，搜索`JReble for Intellij`， 选中安装即可。

&nbsp;

- 最后三种方法是基于类加载机制来实现热加载的，因此你修改完成代码后必须重新编译当前代码，才能触发热部署，Eclipse默认就支持了自动编译，而在**Intellij IDEA中默认是关闭了自动编译的**，可以按照如下2步设置开启：

    - `control + alt + S`打开设置页面，选择`Build,Execut, Deployment -> Compiler` 勾选中左侧的`Build Project automatically`
    - `control + shift + alt + /`打开页面，选择`Registry -> 勾选compiler.automake.allow.when.app.running`即可

&nbsp;
&nbsp;

## 四、Idea常见问题及解决

- **解决Idea中报没有加载到Bean的错误**：

    -  报错：`Could not autowire. No beans of 'xxxx' type found`
    - 解决： https://blog.csdn.net/u012453843/article/details/54906905

- **Idea进行Debug操作：**

    - https://blog.csdn.net/qq_27093465/article/details/64124330

    - 设置断点并运行程序
    ![image](https://user-images.githubusercontent.com/38284818/51900559-9591e300-23f0-11e9-8e72-d285be5d5115.png)
    - **F7**为进入详细方法，及如果改行程序是某一个方法，会进入到方法里面
    - **F8**为直接运行一行代码，即不进入方法中，只运行单行代码
    - **F9**为跳到下一个断点，若无断点，则结束debug





