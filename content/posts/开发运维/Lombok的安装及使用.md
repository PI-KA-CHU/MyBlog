+++
author = "pikachu"
title = "Lombok的安装及使用"
date = "2018-12-29"
description = " "
tags = [
	"开发工具"
]
categories = [
    "it", "开发运维"
]

+++



## Lombok的简介及安装

- Lombok能够简化代码的复杂度，例如对dao层或者bean的类进行**getter，setter，构造方法，toString方法和equal等方法**的进行简化，使用注解即可实现，只要程序实现了该API，就能在**javac**（编译）运行时得到调用，通过注解在编译时生成新的语法树，再进而转换为字节码。

- Lombok插件的下载：https://projectlombok.org/download
（windows环境下）下载插件后，双击lombok.jar进行安装，选择自己的编译器（这里是Eclipse）后install导入即可。
- Lombok的maven依赖：https://projectlombok.org/setup/maven
将maven依赖复制黏贴到项目pom.xml依赖中即可

![lombok-installer](https://user-images.githubusercontent.com/38284818/50521089-ee731080-0afe-11e9-9631-4558fbc647cb.png)


## Lombok的常见注解及使用

- **@Getter/@Setter**：实现get和set方法

- **@ToString**：实现类toString方法

- **@EqualsAndHashCode**：实现hashCode和equals方法

- **@Data**：实现@ToString，@EqualsAndHashCode， @Getter在所有领域，@Setter所有非final字段，以及 @RequiredArgsConstructor方法

- @Slf4j：实现日志创建
`private static final org.slf4j.Logger log = org.slf4j.LoggerFactory.getLogger(LogExample.class);`

- 可使用@Getter(**of = {"id","name"}**)来实现对指定元素构造及@Getter(**exclude = {"id","name"}**)除去选中元素构造

- **官方详细文档**：https://projectlombok.org/features/all

- **使用**：在所需注解类上使用以上注解即可，使用前需导入插件及jar包（或者maven依赖）

![lombok](https://user-images.githubusercontent.com/38284818/50521065-c1bef900-0afe-11e9-9246-6c1230cf51fe.JPG)

