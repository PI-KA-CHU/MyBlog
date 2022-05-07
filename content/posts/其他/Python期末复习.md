+++
author = "pikachu"
title = "Python期末复习"
date = "2019-01-06"
description = "Lorem Ipsum Dolor Si Amet"
tags = [
    "java",
	"复习"
]
categories = [
    "IT",
]
+++


&nbsp;

![default](https://user-images.githubusercontent.com/38284818/50725805-526c8900-113e-11e9-9a4e-1f154620b12f.JPG)

- **列表：** 有序，可变
`[ 'crunchy frog' ,  'ram bladder' ,  'lark vomit' ]`

- **元组：** 有序，不可变
`( 'crunchy frog' ,  'ram bladder' ,  'lark vomit' )`

- **字典：** 无序，可变
`{ 'x' : 'crunchy frog' , 'y' : 'ram bladder' , 'z' : 'lark vomit' }`

- **集合：** 无序，可变
`{ 'crunchy frog' ,  'ram bladder' ,  'lark vomit' }`

- **字符串：** 有序，不可变
`“ crunchy  frog  ram bladder lark  vomit ”`

- **range：** 有序，不可变
`range(10)、range(2, 5)、range(5, 2, -1)`

&nbsp;

**转化：**

- **其他格式转化为list格式：**

![2](https://user-images.githubusercontent.com/38284818/50725829-171e8a00-113f-11e9-9506-78d0719e32bc.png)

- **其他格式转化为元组格式：**

![3](https://user-images.githubusercontent.com/38284818/50725848-695fab00-113f-11e9-9a84-816394eebdb1.png)

- **其他格式转化为字符串格式：** 

> **map语法** ： map(function, iterable)，下面的图中是迭代把i值从nt转化为str，再把list转化为字符串

![4](https://user-images.githubusercontent.com/38284818/50725852-7b414e00-113f-11e9-9e5c-d5b3529aa7ff.png)

- **其他格式转化为字典格式：**

> **enumerate()** 函数用于将一个可遍历的数据对象(如列表、元组或字符串)组合为一个**索引序列**，同时列出数据和数据下标。（使其能够转化为字典）

![5](https://user-images.githubusercontent.com/38284818/50725862-ae83dd00-113f-11e9-9c27-7704f0995834.png)

> **zip(arr)** 函数用于将可迭代的对象作为参数，将对象中对应的元素打包成一个个**元组**，然后返回由这些元组组成的列表。<b>zip(*arr)</b> 是将打包好的重新回到未打包之前的元组列表，相当于**转置**。
如果各个迭代器的元素个数不一致，则 **返回列表长度与最短的对象相同** ，利用 * 号操作符，可以将元组解压为列表。

![6](https://user-images.githubusercontent.com/38284818/50726315-44226b00-1146-11e9-9e2f-c1e1b0483984.png)

- **其他格式转化为集合格式：**

![7](https://user-images.githubusercontent.com/38284818/50726406-957f2a00-1147-11e9-8782-4c4437d10c39.png)

&nbsp;
&nbsp;

## python的有序序列切片

&nbsp;

**切片的格式及使用：**

- 切片使用**2个冒号**分隔的**3个数字**来完成

- 第一个数字表示切片开始位置，默认值为**0**

- 第二个数字表示切片截止（但不包含）位置，默认值为列表长度**len(S)**

- 第三个数字表示切片的步长，默认值为**1**

![8](https://user-images.githubusercontent.com/38284818/50726650-168bf080-114b-11e9-96b4-b7eacacbd591.png)
