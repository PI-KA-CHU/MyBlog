+++
author = "pikachu"
title = "Java Selenium的基本使用"
date = "2019-01-01"
description = " "
tags = [
    "",
	"开发工具"
]
categories = [
    "it","java"
]

+++


## 工具下载

- 下载地址： https://www.seleniumhq.org/download/

- Selenuim使用firefox时所需的驱动： https://github.com/mozilla/geckodriver/releases/tag/v0.9.0

## 基本操作

> 将相关依赖包导入Eclipse中，导入完成后即可使用，下面是**登陆QQ邮箱**的代码：

```
//配置启动Firefox时所需的驱动
System.setProperty("webdriver.gecko.driver",
		"C:\\Users\\pc\\Desktop\\桌面文件夹\\作业\\软件测试\\selenium-jav\\geckodriver.exe");
//配置本地Firefox的地址
System.setProperty("webdriver.firefox.bin", "E:\\Program Files (x86)\\Firefox\\firefox.exe");

WebDriver driver = new FirefoxDriver();

// 设置查找元素最大时间
Timeouts tos = driver.manage().timeouts();
tos.implicitlyWait(5, java.util.concurrent.TimeUnit.SECONDS);

driver.get("https://mail.qq.com"); 
Thread.sleep(3000); 

//跳转到登陆模块
driver.switchTo().frame("login_frame");
//获取账号输入框
WebElement username = driver.findElement(By.name("u")); 
username.sendKeys(usernameValue);
//获取密码输入框
WebElement password = driver.findElement(By.id("p")); 
password.sendKeys(passwordValue);
//模拟点击登陆按钮
WebElement button = driver.findElement(By.id("login_button"));
button.click();

//跳转到上一层界面
driver.switchTo().defaultContent();
//下面是独立密码输入，如果没有的话可以不用
WebElement password2 = driver.findElement(By.id("pp")); 
password2.sendKeys("******");
//模拟点击登陆按钮
WebElement button2 = driver.findElement(By.id("btlogin"));
button2.click();

//获取指定标签的text内容
WebElement logo = driver.findElement(By.id("useraddr"));
String text = logo.getText();
System.out.println(text);

if("529709737@qq.com".equals(text)) {
	System.out.println("页面正确");
        //关闭浏览器
	driver.close();
}


附录：

//通过xpath获取（百度）标签元素：
elm = driver.findElement(By.xpath("//div[@id='1']/h3/a"));

//处理Alert弹出框：
Alert alert = driver.switchTo().alert();
//点击确认
alert.accept();
//点击取消按钮
alert.dismiss()
//需要输入文本的弹出框的处理
alert.sendKeys(keyValue)
//获取弹出框的文本内容
alert.getText()

/*
要在多个窗口之间进行切换，首先必须获取每个窗口的唯一标识符（句柄），
通过WebDriver对象的getWindowHandles()方法，可以获取所有打开窗口的标
识符，并将其以集合的形式返回
*/
String[] handles = new String[driver.getWindowHandles().size()];
driver.getWindowHandles().toArray(handles);

/*
通过Options对象对测试进行设置，设置内容包括Cookie、超时时间和浏览器窗口
Timeouts对象包含三种方法：
1. implicitlyWait：设置脚本在查找元素时的最大时间
2. pageLoadTimeout：页面操作超时时间
3. setScriptTimeout：设置脚本异步执行时间
*/
Timeouts timeouts = driver.manage().timeouts();
Timeouts.implicitlyWait(30, java.util.concurrent.TimeUnit.SECONDS);



```