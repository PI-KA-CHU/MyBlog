

+++

author = "pikachu"
title = "Spring高级应用"
date = "2021-12-07"
description = " "
draft = false
tags = [
	"spring-boot",
	"疑难杂症"
]

categories = [
    "it", "spring"

]

+++

## 

## Spring Boot

1. 当前项目扫描不到通用包的Component

```
解决：在Application类添加包扫描注解

@ComponentScan("com.feida.*")
public class UserManagementServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(UserManagementServiceApplication.class, args);
    }

}
```

