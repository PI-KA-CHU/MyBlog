 +++
author = "pikachu"
title = "Oozie学习（定时任务）"
date = "2019-08-13"
description = "Lorem Ipsum Dolor Si Amet"
draft = false
tags = [
    "大数据"
]
categories = [
    "IT"
]
+++



## 一、Oozie的定时任务方式

### 1.1 官方定义

![image](https://user-images.githubusercontent.com/38284818/62443388-53759f80-b78d-11e9-8e8d-e3cd9f16227c.png)


### 1.2 corntab方式

![image](https://user-images.githubusercontent.com/38284818/62443589-d72f8c00-b78d-11e9-9935-e1b8d103e331.png)

![image](https://user-images.githubusercontent.com/38284818/62443622-ef071000-b78d-11e9-9f77-75cab0326c85.png)


&nbsp;

## 二、编写定时任务

### 2.1 代码编写

**job.properties**
```
nameNode=hdfs://hh:8020
jobTracker=hh:8032
queueName=qcpj_access_log_queue
examplesRoot=accessLogAnalyze

# workflow参数为 oozie.wf.application.path
oozie.coord.application.path=${nameNode}/oozie/workflows/qcpj/${examplesRoot}

start=2019-08-05T23:00+0800
end=2099-08-05T23:30+0800
workflowAppUri=${nameNode}/oozie/workflows/qcpj/${examplesRoot}

```

**workflow.xml**
```
<workflow-app name='qcpj_accessLog_analyze' xmlns="uri:oozie:workflow:0.3">
    <start to="accessLogAnalyze"/>

    <!-- 分析accessLog日志数据 -->
    <action name="accessLogAnalyze">
        <java>
            <job-tracker>${jobTracker}</job-tracker>
            <name-node>${nameNode}</name-node>
            <main-class>com.bnuz.qcpj.mr.accessLogAnalyze.AccessLogRunner</main-class>
        </java>
        <ok to="eventCountRunner"/>
        <error to="kill"/>
    </action>

    <!-- 统计相关事件数量 -->
    <action name="eventCountRunner">
        <java>
            <job-tracker>${jobTracker}</job-tracker>
            <name-node>${nameNode}</name-node>
            <main-class>com.bnuz.qcpj.mr.accessLogEventCount.EventCountRunner</main-class>
        </java>
        <ok to="end"/>
        <error to="kill"/>
    </action>

    <kill name="kill">
        <message>mapreduce failed, error message:${wf:errorMessage(wf:lastErrorNode())}</message>
    </kill>
    <end name="end"/>
</workflow-app>

```

**coordinator.xml**

- 修改时区`timezone`
- 修改版本`xmlns`
```
<coordinator-app name="qcpj_access_log_analyze_coor" frequency="${coord:minutes(5)}" start="${start}" end="${end}" timezone="GMT+0800" xmlns="uri:oozie:coordinator:0.2">
	<action>
		<workflow>
            <app-path>${workflowAppUri}</app-path>
			<configuration>
				<property>
					<name>jobTracker</name>
					<value>${jobTracker}</value>
                </property>
                <property>
                    <name>nameNode</name>
                    <value>${nameNode}</value>
				</property>
				<property>
					<name>queueName</name>
					<value>${queueName}</value>
				</property>
			</configuration>
     		</workflow>
	</action>
</coordinator-app>
```


### 2.2 任务操作命令

（以下命令在任务目录下执行，作业Id可以在Oozie Web主页中查看）
```
# 提交任务（提交后为PRE状态）
oozie job -oozie http://hh:11000/oozie -config job.properties -submit

# 开始任务(指定jobId，PRE -> RUN)
oozie job -oozie http://hh:11000/oozie -start 14-20090525161321-oozie-joe

# 暂停任务（RUN -> SUSPEND）
oozie job -oozie http://hh:11000/oozie -suspend 14-20090525161321-oozie-joe

# 恢复任务（SUSPEND -> RUN）
oozie job -oozie http://hh:11000/oozie -resume 14-20090525161321-oozie-joe

# 重新运行工作流（仅限工作流）
oozie job -oozie http://hh:11000/oozie -config job.properties -rerun 14-20090525161321-oozie-joe -D oozie.wf.rerun.failnodes=false

# 重新运行定时任务（action表示如下图 future-1）
oozie job -oozie http://hh:11000/oozie -config job.properties -rerun 14-20090525161321-oozie-joe -refresh -action 1-4

# 提交并启动任务（RUN状态）
oozie job -oozie http://hh:11000/oozie -config job.properties -run

# 杀死指定任务（KILL状态）
oozie job -oozie http://hh:11000/oozie -kill 0000358-190502173832787-oozie-hado-C


# 查看工作流信息
oozie job -oozie http://hh:11000/oozie -config job.properties -info 14-20090525161321-oozie-joe

# 查看工作流日志
oozie job -oozie http://hh:11000/oozie -config job.properties -log 14-20090525161321-oozie-joe

# 
```

future-1

![image](https://user-images.githubusercontent.com/38284818/62447540-34c8d600-b798-11e9-908b-71936e6c46c3.png)

&nbsp;

## 三、参考

- https://www.cnblogs.com/30go/p/8391328.html
- 定时任务： https://www.cnblogs.com/cenzhongman/p/7259226.html
- 官方文档： https://oozie.apache.org/docs/4.0.0/DG_CommandLineTool.html