 +++
author = "pikachu"
title = "Oozie学习（基本任务）"
date = "2019-08-02"
description = "Lorem Ipsum Dolor Si Amet"
draft = false
tags = [
    "大数据"
]
categories = [
    "IT"
]
+++



## 一、oozie组件

- `Workflow`：工作流（流程图）
    - 常用结点：
        -  控制流结点（Control Flow Nodes）
            -  start、end、kill、分支、合并等
        -  动作结点（Action Nodes）
- `Coordinator`：定时器
- `Bundle Job`：绑定多个Coordinator

<br>
<br>

## 二、环境搭建

- 学习网站： https://www.bilibili.com/video/av35357845/?p=4&spm_id_from=333.788.b_6d756c74695f70616765.4

- 下面记录的是一些零碎的过程
    -   将依赖的Lib上传到Hdfs中
    -   创建Oozie.sql文件，运行该文件自动生成表（可以通过Navicat查看）
    -   启动Oozie `bin/oozied.sh start`
    -   `jps -l`：查看运行的Java类，其中`Bootstrap`表明`Oozie`启动成功

<br>
<br>

## 三、Oozie shell的编写及启动：

- 工作流程
    - 编写`job.properties`及`workflow.xml`
    - 将文件上传到Haoop的HDFS中
    - 通过命令执行oozie任务
    - 查看oozie的执行页面：http://hh:11000/oozie/
<br>

**文件编写：**

- job.poperties配置：
    - 指定`NameNode`（HDFS）和`JobTracker`（Yarn）
    - 指定`workflow`在HDFS中的路径
```
nameNode=hdfs://hh:8020     # hdfs地址
jobTracker=hh:8032          # yarn地址
queueName=default           
examplesRoot=shell

oozie.wf.application.path=${nameNode}/oozie/workflows/example/oozie-example/apps/${examplesRoot}/                    # workflow上传到HDFS后的地址
```
- workflow.xml配置（官方例子）：
    - 注意下面配置中，`xmlns`版本与`oozie-site.xml`的版本需要一致，否则会报无法识别的错误
```
<workflow-app xmlns="uri:oozie:workflow:0.3" name="shell-wf">
    <start to="shell-node"/>
    <action name="shell-node">
        <shell xmlns="uri:oozie:shell-action:0.1">
            <job-tracker>${jobTracker}</job-tracker>
            <name-node>${nameNode}</name-node>
            <configuration>
                <property>
                    <name>mapred.job.queue.name</name>
                    <value>${queueName}</value>
                </property>
            </configuration>
            <exec>echo</exec>
            <argument>my_output=Hello Oozie</argument>
            <capture-output/>
        </shell>
        <ok to="check-output"/>
        <error to="fail"/>
    </action>
    <decision name="check-output">
        <switch>
            <case to="end">
                ${wf:actionData('shell-node')['my_output'] eq 'Hello Oozie'}
            </case>
            <default to="fail-output"/>
        </switch>
    </decision>
    <kill name="fail">
        <message>Shell action failed, error message[${wf:errorMessage(wf:lastErrorNode())}]</message>
    </kill>
    <kill name="fail-output">
        <message>Incorrect output, expected [Hello Oozie] but was [${wf:actionData('shell-node')['my_output']}]</message>
    </kill>
    <end name="end"/>
</workflow-app>
```

<br>

**文件上传**

- `hadoop dfs -put /home/hadoop/oozie-example/apps/shell/* /oozie/workflows/example/oozie-example/apps/shell`

![image](https://user-images.githubusercontent.com/38284818/61775204-09a2c600-ae2b-11e9-9f02-6c7ca55e423d.png)

<br>

**执行作业**

- `oozie job -oozie http://hh:11000/oozie -config job.properties -run`
    - 其中`job.properties`为本地的文件，而不是HDFS上的

<br>

**查看作业**

- Oozie访问页面：`http://hh:11000/oozie/`
![image](https://user-images.githubusercontent.com/38284818/61775639-e4628780-ae2b-11e9-95ba-931da466633a.png)
- Yarn访问页面：`http://172.20.13.20:8088/cluster`
![image](https://user-images.githubusercontent.com/38284818/61775761-1bd13400-ae2c-11e9-8a6a-26f934a3e8ea.png)