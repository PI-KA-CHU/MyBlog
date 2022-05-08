+++
author = "pikachu"
title = "Hadoop平台的搭建"
date = "2019-03-12"
description = " "
draft = false
tags = [
	"大数据"
]
categories = [
    "it"
]

+++

## 一、搭建步骤

#### 1. 服务器（Linux）环境准备
- 修改系统文件需要root权限，可通过以下命令获取

```
su root
（回车后输入密码）
```

- 配置服务器的主机名称（**CentOS 7的坑**，需要修改`/etc/hostname`文件主机名才会修改有效，重启后生效）

```
vi /etc/hostname
node-1（修改的主机名）
```

- 配置服务器IP地址和主机名的映射

```
vi /etc/hosts
192.168.125.129 node-1
192.168.125.130 node-2
192.168.125.131 node-3
```

- 配置主服务器（namenode所在服务器）的ssh免密登陆

```
# 生成ssh免登陆密钥
ssh-keygen -t rsa（输入命令后四个回车）

# 将公钥拷贝到免密登陆的目标机器上
ssh-copy-id node-2（node-2为目标服务器）

# 配置成功后，可通过以下命令免密码登陆
ssh node-2（目标主机名）

# 退出ssh连接的主机
exit
```

- 关闭服务器防火墙（CentOS 7）

```
# 查看防火墙状态
systemctl status firewalld.service

# 关闭防火墙
systemctl stop firewalld.service

# 禁止防火墙开机自启动
systemctl disable firewalld.service
```

#### 2. 删除本机jdk并配置sun公司的jdk

- 将windows下载好的jdk压缩包**复制粘贴**到linux服务器中（**不要使用拖拽**，否则解压会出错），然后执行下面命令

```
# 删除本机的openjdk
rpm -qa|grep java
rpm -e --nodeps xxxxxx（要删除的包名）

# 解压下载的 jdk
tar zxvf 压缩包地址 -C 解压地址
```

- 配置环境变量 /etc/profile
```
export JAVA_HOME=/usr/java/jdk1.8.0_191 (等号右边为解压的jdk目录)
export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$JAVA_HOME/bin:$PATH
```
- 使配置的环境变量生效（谨慎设置，否则系统会无法正常启动
`source /etc/profile`

#### 3. 下载hadoop压缩包并解压
- 下载地址：https://hadoop.apache.org/releases.html（本人下载的是2.7.7的二进制版本，需要的话可以下载源码在服务器自行编译）
- 解压命令：`tar -zxvf hadoop-2.7.7.tar.gz /usr/hadoop/`
- 修改解压后的hadoop的etc目录下的相关配置：
    - `hadoop-env.sh`：加入jdk环境变量：
    ```
    # The java implementation to use.
    export JAVA_HOME=/usr/java/jdk1.8.0_191
    ```

    - `core-size.xml`：
    ```
    <configuration>
    <!-- 指定Hadoop使用的文件系统schema（URI），HDFS的老大（NameNode）所在的地址  -->
	<property>
		<name>fs.defaultFS</name>
		<value>hdfs://node-1:9000</value>
	</property>
	<!-- 指定hadoop运行时产生的存储目录 -->
	<property>
		<name>hadoop.tmp.dir</name>
		<value>/usr/hadoop/tmp</value>
	</property>
    </configuration>
    ```
    - `hdfs-site.xml`:
    ```
    <configuration>
    <!-- 指定HDFS副本数量 -->
	<property>
	     <name>dfs.replication</name>
	     <value>2</value>
	</property>
	<!-- 指定HDFS的秘书节点的地址 -->
	<property>
	     <name>dfs.namenode.secondary.http-address</name>
	     <value>node-2:50090</value>
	</property>
    </configuration>
    ```
    - `mapred-site.xml`：
    ```
    <configuration>
    <!-- 指定mapreduce运行在yarn上，默认是local模拟 -->
	<property>
		<name>mapreduce.framework.name</name>
		<value>yarn</value>
	</property>
    </configuration>
    ```
    - `yarn-site.xml`：
    ```
    <configuration>
    <!-- 指定YARN的老大（ResourceManager）的地址 -->
    <property>
	     <name>yarn.resourcemanager.hostname</name>
	     <value>node-1</value>
	</property>
    
    <!-- NodeManager上运行的附属服务。需要配置成mapreduce_shuffle，才可MapReduce程序默认值 -->
    <property>
	     <name>yarn.nodemanager.aux-services</name>
	     <value>mapreduce_shuffle</value>
	</property>
	</configuration>
    ```
    - `slaves`：(默认为localhost)
    ```
    <!-- 指定从节点所在的主机名，启动DataNode -->
    node-1
    node-2
    node-3
    ```
    - `/etc/profile`：修改系统环境变量(终端按G可以到达文件末尾)
    ```
    export HADOOP_HOME=/usr/hadoop/hadoop-2.7.7
    export PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
    
    # 修改完后执行命令 `resource /etc/profile`使得环境变量生效
    ```

#### 4. 同步到其他服务器
- **使用远程命令将配置文件发送到各个服务器上**：
    - `scp -r /usr/hadoop/hadoop-2.7.7/ root@node-2:/usr/hadoop/`(r表示递归传递)
- **将环境变量文件发送到其他服务器**：
    - `scp -r /etc/profile root@node-2:/etc/`
    - 发送完成后执行`source /etc/profile`

#### 5. 格式化NameNode
- 首次启动HDFS时，需要对其进行**格式化操作**
- 格式化本质是进行文件系统的**初始化操作**，创建一些自己所需的文件
- 格式化操作成功后，后续不要再进行格式化，否则可能导致集群启动失败
- **注意**：需要在**root权限或者sudo权限**下执行格式化命令，否则会格式异常
```
hdfs namenode -format
```

#### 6. 启动集群
- 启动`hdfs集群`(在hadoop的sbin目录下执行下面命令)
```
start-dfs.sh
```
- 启动`yarn集群`(在hadoop的sbin目录下执行下面命令)
```
start-yarn.sh
```
- 使用`jps`命令查看是否启动成功
![image](https://user-images.githubusercontent.com/38284818/54256844-4b2b8680-4598-11e9-8379-617faa843b99.png)

#### 7. 测试集群是否可用
- 集群启动成功后可打开下面网址（node-1为主节点所在主机名）：
    - 打开**hdfs**网址（NameNode所在服务器）：http://node-1:50070
    - 打开**yarn**网址（ResourceManager所在服务器）：http://node-1:8088

- **初试hdfs**：（执行如下命令）
```
# 创建hello文件夹
hdfs dfs -mkdir /hello

# 上传文件到hadoop
hdfs dfs -put /指定文件或者文件夹
（或者 hadoop fs -put /指定文件或者文件夹 ）

# 将文件下载到本地
hdfs dfs -get /hadoop中存储的文件的目录
(或者 hadoop fs -get /hadoop中存储的文件的目录)

# 获取hadoop文件目录下的文件
hdfs dfs -ls /指定文件或者文件夹
（或者 hadoop fs -ls /指定文件或者文件夹 ）
```
- 初试**mapreduce**：(测试文件在hadoop安装目录/share/hadoop/mapreduce/下)
```
# mapreduce执行圆周率运算
hadoop jar hadoop-mapreduce-examples-2.7.7.jar pi 20 50
```

#### Hadoop Shell 命令
- http://hadoop.apache.org/docs/r1.0.4/cn/hdfs_shell.html

&nbsp;

## 二、问题与解决

#### BUG

- 配置成功后如果出现ping不通其他服务器的情况，则检查下服务器的网络是否已经连接
- 在`/etc/hosts`加入IP地址和主机名的映射的时候，除了加入`IP 主机名`不要修改其他地方，否则可能出现ping不通其他服务器的情况
- 环境变量配置错误，把整个三个虚拟机的系统搞炸了！！垃圾BD
    - 解决方法：在登陆界面处`control + alt + F2`进入终端，以root方式登陆，然后通过命令`/usr/bin/vi /etc/profile`将错误的环境变量删除，然后`reboot`重启系统即可恢复（差点绝望）
    - 参考：https://blog.csdn.net/ysy950803/article/details/60777802
