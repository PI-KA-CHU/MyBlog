+++
author = "pikachu"
title = "MySQL语句学习"
date = "2018-10-26"
description = "Lorem Ipsum Dolor Si Amet"
tags = [
    "mysql"
]
categories = [
    "IT",
]
+++


## 一、Insert语句


#### insert into和replace into以及insert ignore用法区别

- <i>**insert into**</i>表示插入数据，数据库会检查主键，如果出现重复会报错； 

- <i>**replace into**</i>表示插入替换数据，需求表中有<b>PrimaryKey</b>，或者<b>unique</b>索引，如果数据库已经存在数据，则用新数据替换，如果没有数据效果则和insert into一样； 

- <i>**insert ignore**</i>表示，如果中已经存在相同的记录，则忽略当前新数据；

&nbsp;


## 二、日期转化为星期的SQL语句

```
SELECT
	CASE dayofweek(‘2018-12-17’)
WHEN 1 THEN
	'星期日'
WHEN 2 THEN
	'星期一'
WHEN 3 THEN
	'星期二'
WHEN 4 THEN
	'星期三'
WHEN 5 THEN
	'星期四'
WHEN 6 THEN
	'星期五'
WHEN 7 THEN
	'星期六'
END
FROM
	DUAL
```

&nbsp;

## 三、自定义排序（如中文排序）

- **根据自定义的排序进行数据排序获取**

```
SELECT
	*
FROM
	classes
WHERE
	gradeId = "44040012"
ORDER BY
	instr(
		"一班,二班,三班,四班,五班,六班,七班,八班,九班,十班,十一班,十二班",
		className
	)
```