+++
author = "pikachu"
title = "JUnit单元测试的使用"
date = "2018-10-10"
description = "Lorem Ipsum Dolor Si Amet"
tags = [
    "java"
]
categories = [
    "IT",
]

+++


#### 关键代码：
【配置好spring及JUnit的class（创建一次即可，后续的可以直接继承）】

```
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration({ "classpath:spring/spring-dao.xml", "classpath:spring/spring-service.xml" })
public class BaseTest {
}
```
&nbsp;

#### 创建步骤：

1. 右键点击想测试的类（或包）,选择New-->Oher：

![20130730143840531](https://user-images.githubusercontent.com/38284818/46738341-8582b600-ccd0-11e8-9f70-c527a5a0a56e.png)
<br/>

2. 找到JUnit Test Case，创建：

![20130730144221390](https://user-images.githubusercontent.com/38284818/46739475-3722e680-ccd3-11e8-9fbb-27cbf5b86f40.png)
<br/>

3. 选择JUnit 4 Test，输入名称(Name)，命名规则一般建议采用：**类名+Test**

![20130730145611296](https://user-images.githubusercontent.com/38284818/46739798-eb247180-ccd3-11e8-8618-23c51df4db4a.png)
<br/>

4. 勾选自己要测试的方法：

![20130730145746000](https://user-images.githubusercontent.com/38284818/46739998-5bcb8e00-ccd4-11e8-986f-24307ae07b9d.png)
<br/>

5. 生成代码，在要测试的方法中写入代码

![20130730150135234](https://user-images.githubusercontent.com/38284818/46739906-27f06880-ccd4-11e8-9632-36d58d37a9ef.png)
<br/>
