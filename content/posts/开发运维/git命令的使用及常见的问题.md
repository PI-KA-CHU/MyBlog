+++
author = "pikachu"
title = "git命令的使用及常见的问题"
date = "2019-02-24"
description = " "
tags = [
	"开发工具"
]
categories = [
    "it", "开发运维"
]

+++


#### 获取git仓库

```
git init  //初始化本地仓库（会自动创建一个名为 .git 的子目录）
git clone [URL]  //拉取远程代码到本地
```


#### git提交代码的基本操作

```
git add .  // 提交所有修改的文件
git commit -m "提交的版本说明"  // 将版本提交到本地仓库
git push   // 将代码
```


#### 常见异常及解决

**操作**：
使用`git pull`命令，`git pull = git fetch + merge`

**异常**：
```
Pull is not possible because you have unmerged files.
Please, fix them up in the work tree, and then use 'git add/rm <file>'
as appropriate to mark resolution, or use 'git commit -a'.
```
**解决**：
```
git fetch origin  // 拿到了远程所有分支的更新
git reset --hard origin/master  // 完全舍弃你没有提交的改动
git pull  // 拉取远程项目
```


#### 修改`.gitignore`文件忽略上传的文件
- 下面的结果是：忽略除了src、bin文件夹及pom.xml文件的其他所有文件，及除了这些其他文件不会被跟踪上传

```
/*    # 忽略所有文件
!/src    # 忽略除了src文件夹的其他文件
!/bin    # 忽略除了bin文件夹的其他文件
!/pom.xml    #忽略除了pom.xml文件的其他文件
```



#### 添加目标git项目作为submodule到public目录中

```
git submodule add https://github.com/PI-KA-CHU/PI-KA-CHU.github.io.git public

问题：
'public' already exists in the index

解决：
1. git ls-files --stage public
2. 看到 160000开头的目录
3. git rm --cached public
4. 重新添加
```



#### 重置本地修改

```
# 忽略本地修改，使用远方分支
git reset --hard origin/master
```



