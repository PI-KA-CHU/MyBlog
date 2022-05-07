+++

author = "pikachu"
title = "MySQL执行一条语句的过程"
date = "2022-03-10"
description = " InnoDB存储引擎，redo log，undo log，bin log"
draft = true
tags = [
    "java", "mysql"
]
categories = [
    "IT","数据库"
]

+++

- https://www.pdai.tech/md/db/sql-mysql/sql-mysql-execute.html







### bin log三种模式的区别

- https://blog.csdn.net/keda8997110/article/details/50895171

1. **row**：记录具体的行被修改的内容。优点是记录清晰，复制完整；缺点是日志量较大。
2. **statement**：记录的是执行的SQL语句。优点是日志量少；缺点是对于某些特定的函数（sleep()、last_insert_id())，可能由于版本问题导致复制失败。
3. **mixed**：混合模式，对于正常的情况采用statement模式，对于statement模式无法复制的操作则采用row模式。