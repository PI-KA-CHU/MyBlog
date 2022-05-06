+++
author = "pikachu"
title = "SpringBoot整合Junit4"
date = "2018-11-21"
description = "Lorem Ipsum Dolor Si Amet"
tags = [
    "java",
	"spring-boot"
]
categories = [
    "IT",
]
+++



Junit单元测试在SSM中的使用已经有简单介绍：#4，基本的使用是一致的，但是在配置上有所不同：

&nbsp;


**依赖包：**

```
<dependency>
        <groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-test</artifactId>
	<scope>test</scope>
</dependency>
	<dependency>
	<groupId>com.jayway.jsonpath</groupId>
	<artifactId>json-path</artifactId>
</dependency>
```

**注释配置：**（其中**LostBackSysApplication**为自己的启动项，即main函数中启动的那个类）
```
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest(classes = LostBackSysApplication.class)
public class FormsDaoTest {

	@Autowired
	private FormsDao formsDao;
	
	@Before
	public void setUp() throws Exception {
	}

	@Test
	public void testInsertApplicationFrom() {
		ApplicationFormDto applicationFormDto = new ApplicationFormDto();
		applicationFormDto.setId(UUID.randomUUID().toString());
		applicationFormDto.setUserID("ed8e29c5-4684-4fae-967b-271c71d217ac");
		applicationFormDto.setPickedItemID("6ad6152b-e7d5-42d4-8939-53b3ce79da8b");
		applicationFormDto.setCreateTime("2018-11-21");
		applicationFormDto.setDescription("这个是我的");
		
		formsDao.insertApplicationFrom(applicationFormDto);
	}

}
```