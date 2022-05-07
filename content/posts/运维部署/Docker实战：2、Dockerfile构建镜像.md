+++
author = "pikachu"
title = "Docker实战：2、Dockerfile构建镜像"
date = "2019-08-20"
description = "Lorem Ipsum Dolor Si Amet"
tags = [
	"docker",
	"开发工具"
]
categories = [
    "IT"
]
+++



## 一、概念

> 镜像的定制实际就是定制每一层所添加的配置、文件。如果我们把每一层修改、安装、构建、操作的命令都写入一个脚本，用这个脚本来构建、定制镜像，那么就可以实现快速的搭建一个服务环境并快速配置环境相关变量，而不是靠手动的搭建及命令行操作，这个脚本就是Dockerfile。


#### 1.1、文件格式：(四部分)

- 基础镜像信息
- 维护者信息
- 镜像操作指令
- 容器启动执行指令

```
# 1、第一行必须指定 基础镜像信息
FROM ubuntu

# 2、维护者信息
MAINTAINER docker_user docker_user@email.com

# 3、镜像操作指令
RUN echo "deb http://archive.ubuntu.com/ubuntu/ raring main universe" >> /etc/apt/sources.list
RUN apt-get update && apt-get install -y nginx
RUN echo "\ndaemon off;" >> /etc/nginx/nginx.conf

# 4、容器启动执行指令
CMD /usr/sbin/nginx
```

&nbsp;

#### 1.2、常用命令


**镜像构建**：(`-f`指定Dockerfile路径，`.`表示当前路径)
```
docker build .
docker build -f /path/to/a/Dockerfile .
```

**镜像标签**：(`-t`构建 仓库/标签，可以支持多个)
```
docker build -t nginx/v3 .
docker build -t nginx/v3:1.0.2 -t nginx/v3:latest .
```

&nbsp;

#### 1.3、指令格式及说明


(1)、FROM（指定基础image）
```
FROM <image>

# 指定版本
FROM <image>:<tag>
```

(2)、MAINTAINER（用来指定镜像`创建者信息`）
```
MAINTAINER <name>
```

(3)、RUN（安装软件用）

- 构建指令，RUN可以运行任何被基础image支持的命令。如基础image选择了ubuntu，那么软件管理部分只能使用ubuntu的命令。
- RUN命令将在当前image中执行任意合法命令并提交执行结果。命令执行提交后，就会自动执行Dockerfile中的`下一个指令`。
- RUN 指令缓存不会在下个命令执行时自动失效。比如 RUN apt-get dist-upgrade -y 的缓存就可能被用于下一个指令. --no-cache 标志可以被用于强制取消缓存使用。
```
RUN <command> (the command is run in a shell - /bin/sh -c)
RUN ["executable", "param1", "param2" ... ] (exec form)
```

(4)、CMD（设置`container启动`时执行的`操作`）

- 设置指令，用于`container启动时`指定的操作。该操作可以是执行自定义脚本，也可以是执行系统命令。该指令只能在文件中存在一次，如果有多个，则`只执行最后一条`。
```
# （三种格式）
CMD ["executable","param1","param2"] (like an exec, this is the preferred form)
CMD command param1 param2 (as a shell)

# 当Dockerfile指定了ENTRYPOINT，那么使用下面的格式：
CMD ["param1","param2"] (作为ENTRYPOINT的默认参数)
```
(5)、ENTRYPOINT（设置`container启动时`执行的`操作`）
```
# 单独使用
ENTRYPOINT ["executable", "param1", "param2"] (like an exec, the preferred form)
ENTRYPOINT command param1 param2 (as a shell)

# 于CMD结合使用 - 2种情况
# a、CMD为完整的可执行命令，则指令将不会被执行，只有ENTRYPOINT指令被执行（相互覆盖）
CMD echo “Hello, World!”  
ENTRYPOINT ls -l  

# b、CMD是参数部分，ENTRYPOINT指令只能使用JSON方式指定执行命令，而不能指定参数
FROM ubuntu  
CMD ["-l"]  
ENTRYPOINT ["/usr/bin/ls"]  
```
(6)、USER（设置container容器的`用户`）

- 设置指令，设置启动容器的用户，默认是root用户。
```
# 指定memcached的运行用户  
ENTRYPOINT ["memcached"]  
USER daemon  
或  
ENTRYPOINT ["memcached", "-u", "daemon"]  
```
(7)、EXPOSE（指定容器需要`映射`到宿主机器的`端口`）

- 设置指令，该指令会将容器中的端口映射成宿主机器中的某个端口。当你需要访问容器的时候，可以不是用容器的IP地址而是使用宿主机器的IP地址和映射后的端口。

    - 一、在Dockerfile使用EXPOSE设置需要映射的容器端口
    - 二、在运行容器的时候指定-p选项加上EXPOSE设置的端口
```
# 注：若无指定宿主主机的映射端口，则会被随机映射

# 映射一个端口  
EXPOSE port1  
# 相应的运行容器使用的命令  
docker run -p port1 image  

# 映射多个端口  
EXPOSE port1 port2 port3  
# 相应的运行容器使用的命令（随机）
docker run -p port1 -p port2 -p port3 image  
# 指定需要映射到宿主机器上的某个端口号  （指定）
docker run -p host_port1:port1 -p host_port2:port2 -p host_port3:port3 image  
```

- 端口映射是docker比较重要的一个功能，原因在于我们`每次运行容器的时候容器的IP地址不能指定，而是在桥接网卡的地址范围内随机生成的`。宿主机器的IP地址是固定的，我们可以将容器的端口的映射到宿主机器上的一个端口，免去每次访问容器中的某个服务时都要查看容器的IP的地址。对于一个运行的容器，可以使用`docker port + 容器中需要映射的端口和容器的ID`来查看该端口号在宿主机器上的映射端口。

(8)、ENV（用于设置`环境变量`）

- ENV设置的环境变量，可以使用`docker inspect`命令来查看。同时还可以使用`docker run --env <key>=<value>`来修改环境变量。
```
ENV <key> <value>
# 例:
ENV JAVA_HOME /path/to/java/dirent
```

(9)、ADD（从src`复制文件`到container的dest路径）

- 所有拷贝到container中的文件和`文件夹权限为0755`，uid和gid为0
- 如果文件是可识别的压缩格式，则docker会`自动解压缩`（注意压缩格式）
- 如果<src>是文件且<dest>中`不使用`斜杠结束，则会将<dest>视为`文件`，<src>的内容会写入<dest>
- 如果<src>是文件且<dest>中`使用斜杠`结束，则会<src>文件拷贝到<dest>目录下。
```
# <src> 是相对被构建的源目录的相对路径，可以是文件或目录的路径，也可以是一个远程的文件url;
# <dest> 是container中的绝对路径
ADD <src> <dest>
```

(10)、VOLUME (指定挂载点)

- 创建一个可以从本地主机或其他容器挂载的挂载点，一般用来存放数据库和需要保持的数据等。
- Volume设置指令，使容器中的一个目录具有`持久化存储数据`的功能，该目录可以被容器本身使用，也可以共享给其他容器使用。我们知道容器使用的是`AUFS`，这种文件系统不能持久化数据，`当容器关闭后，所有的更改都会丢失`。当容器中的应用有持久化数据的需求时可以在Dockerfile中使用该指令。如MySQL数据、日志数据等如果不挂载到宿主机器，重启容器后数据都会消失。
```
VOLUME ["<mountpoint>"]

# 例：
VOLUME ["/tmp/data"]
```
- 运行通过该Dockerfile生成image的容器，/tmp/data目录中的数据在容器关闭后，里面的数据还存在。例如另一个容器也有持久化数据的需求，且想使用上面容器共享的/tmp/data目录，那么可以运行下面的命令启动一个容器：
```
# container1为第一个容器的ID，image2为第二个容器运行image的名字。
docker run -t -i -rm -volumes-from container1 image2 bash
```

(11)、WORKDIR（切换目录）

- 设置指令，可以多次切换(相当于`cd命令`)，对RUN,CMD,ENTRYPOINT生效。
```
WORKDIR /path/to/workdir

# 例:在 /p1/p2 下执行 vim a.txt  
WORKDIR /p1
WORKDIR p2
RUN vim a.txt  

```

(12)、ONBUILD（在子镜像中执行）

- ONBUILD 指定的命令在构建镜像时并不执行，而是在它的子镜像中执行。
```
ONBUILD <Dockerfile关键字>
```

(13)、COPY(复制本地主机的src文件为container的dest)

- 复制本地主机的src文件（为Dockerfile所在目录的相对路径、文件或目录 ）到container的dest。目标路径不存在时，会自动创建。
```
# 当使用本地目录为源目录时，推荐使用COPY
COPY <src> <dest>
```

(14)、ARG(设置构建镜像时变量)

- ARG指令在Docker1.9版本才加入的新指令，ARG 定义的变量`只在建立 image 时有效`，建立完成后变量就失效消失
```
ARG <key>=<value>
```

(15)、LABEL(定义标签)

- 定义一个 image 标签 Owner，并赋值，其值为变量 Name 的值。
```
LABEL Owner=$Name
```

&nbsp;
&nbsp;

## 二、实战（简单版 - nginx为例）

**2.1、创建Dockerfile文件**

- FROM：选择指定基础镜像
- RUN：执行指令
```
FROM nginx
RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
```


**2.2、运行Dockerfile构建镜像**
```
# 构建镜像
docker build -t nginx:v1 .

# 查看镜像
docker images
```


**2.3、启动镜像容器**
```
# --name 为容器名，-d为后台运行，-p为指定映射端口:端口，nginx:v1为镜像:标签
docker run --name docker_nginx_v1 -d -p 80:80 nginx:v1
```
- *问题一*：主机无法访问虚拟机nginx
- *解决*：防火墙配置问题
    - https://blog.csdn.net/harris135/article/details/74167910

- *问题二*：无法正常启动docker实例
```
(iptables failed: iptables --wait -t nat -A DOCKER -p tcp -d 0/0 --dport 8801 -j DNAT --to-destination 172.17.0.3:80 ! -i docker0: iptables: No chain/target/match by that name.
```
- *解决*：重启docker
```
systemctl restart docker
```

&nbsp;
&nbsp;

## 三、修改容器内容

- 在容器启动后，需要对里面的文件进行修改，可以使用`docker exec -it xx bash`命令再次进行修改（此处修改nginx的主页面显示）
```
docker exec -it docker_nginx_v1 bash
root@3729b97e8226: echo '<h1>Hello, Docker neo!</h1>' > /usr/share/nginx/html/index.html
root@3729b97e8226: exit
exit
```

- （容器的存储层）修改后，可以通过 `docker diff` 命令看到具体的改动
```
docker diff docker_nginx_v1
```

&nbsp;


## 四、SpringBoot整合Docker

- 在已经搭建好SpringBoot的基础上


#### 4.1、项目docker支持准备


a、在pom.xml文件中添加docker支持
```
<properties>
	<docker.image.prefix>springboot-docker</docker.image.prefix>
</properties>

<!-- docker插件支持 -->
<plugin>
	<groupId>com.spotify</groupId>
	<artifactId>docker-maven-plugin</artifactId>
	<version>1.0.0</version>
	<configuration>
	    <!-- 注意artifactId不能包含大写字母 -->
		<imageName>${docker.image.prefix}/${project.artifactId}</imageName>
		<dockerDirectory>src/main/docker</dockerDirectory>
		<resources>
			<resource>
				<targetPath>/</targetPath>
				<directory>${project.build.directory}</directory>
                <include>${project.build.finalName}.jar</include>
			</resource>
		</resources>
	</configuration>
</plugin>
```

b、在项目的`src/main/docker`目录下创建`Dockerfile`文件：
```
FROM openjdk:8-jdk-alpine
VOLUME /tmp
ADD bigdata_presentation.jar app.jar
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
```
**参数解释**：

- **FROM**：表示使用 Jdk8 环境 为基础镜像，如果镜像不是本地的会从 DockerHub 进行下载

- **VOLIME**：VOLUME 指向了一个/tmp的目录，由于 Spring Boot 使用内置的Tomcat容器，Tomcat 默认使用/tmp作为工作目录。这个命令的效果是：在宿主机的/var/lib/docker目录下创建一个临时文件并把它链接到容器中的/tmp目录

- **ADD**：拷贝文件并且重命名

- **ENTRYPOINT**：为了缩短 Tomcat 的启动时间，添加java.security.egd的系统属性指向/dev/urandom作为 ENTRYPOINT

&nbsp;

#### 4.2、项目打包环境构建


**a、dcker安装**
```
请参考上一篇
```

**b、JDK环境安装**
```
# 安装
yum -y install java-1.8.0-openjdk*

# 修改环境变量（/etc/profile）
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.161-0.b14.el7_4.x86_64
export PATH=$PATH:$JAVA_HOME/bin

# 使环境变量生效
source /etc/profile

# 检查是否安装成功
java -version
```

**c、Maven环境安装**
```
# 下载压缩包
wget http://mirrors.shu.edu.cn/apache/maven/maven-3/3.1.1/binaries/apache-maven-3.1.1-bin.tar.gz

# 解压
tar zxvf apache-maven-3.5.2-bin.tar.gz
# 移动
mv apache-maven-3.5.2 /usr/local/maven3

# 修改环境变量（/etc/profile）
MAVEN_HOME=/usr/local/maven3
export MAVEN_HOME
export PATH=${PATH}:${MAVEN_HOME}/bin

# 使环境变量生效
source /etc/profile

# 修改maven为阿里云镜像仓库，否则镜像构建的时候会很慢
自行百度

# 检测是否安装成功
mvn -version
```

&nbsp;

#### 4.3、构建SpringBoot的docker镜像
> 将SpringBoot项目拷贝到服务器中，并通过命令行进入其项目目录

a、测试SpringBoot环境是否正常
```
# 打包
mvn clean package -Dmaven.skip.test=true

# 启动
java -jar target/spring-boot-docker-1.0.jar
```

b、SpringBoot正常则进行其docker镜像的构建
```
# 构建docker镜像
mvn package docker:build

# 查看构建成功的镜像
docker images

# 运行镜像容器
docker run -p 8080:8080 -t springboot-docker/bigdata_presentation

# 查看正运行的容器（-a 可查看所有）
docker ps

# 构建成功如下
...
Building image springboot-docker/bigdata_presentation
Step 1/4 : FROM openjdk:8-jdk-alpine

 ---> a3562aa0b991
Step 2/4 : VOLUME /tmp

 ---> Using cache
 ---> b7ebabcea704
Step 3/4 : ADD bigdata_presentation.jar app.jar

 ---> ed2bf2401a6f
Step 4/4 : ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]

 ---> Running in 718ca03aaa8f
Removing intermediate container 718ca03aaa8f
 ---> 4f1bedaaf992
ProgressMessage{id=null, status=null, stream=null, error=null, progress=null, progressDetail=null}
Successfully built 4f1bedaaf992
Successfully tagged springboot-docker/bigdata_presentation:latest
[INFO] Built springboot-docker/bigdata_presentation
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 18.061s
[INFO] Finished at: Tue Aug 20 11:54:44 CST 2019
[INFO] Final Memory: 43M/103M
[INFO] ------------------------------------------------------------------------

```
访问首页：
![image](https://user-images.githubusercontent.com/38284818/63319679-71c2ca00-c34d-11e9-9110-bd22dc350031.png)

&nbsp;

#### 4.4、问题及解决


**问题一**：springBoot执行mvn命令异常，Spring Boot:jar中没有主清单属性

**解决**：
> 参考：https://blog.csdn.net/u010429286/article/details/79085212

**问题二**（坑爹）：mvn构建docker镜像时候异常


>  I/O exception (java.io.IOException) caught when processing request to {}->unix://localhost:80: Connection reset by peer

**解决**：
> pom文件中的artifactId不能为大写的，否则会出现上面的异常（坑爹）
参考：https://blog.csdn.net/EasternUnbeaten/article/details/79825851

**问题三**：如何自定义maven打包后的文件名

**解决**：
```
在pom文件的<build></build>中加入<finalName>自定义文件名</finalName>
```

&nbsp;

## 五、参考
- http://www.ityouknow.com/docker/2018/03/12/docker-use-dockerfile.html
- http://www.ityouknow.com/docker/2018/03/15/docker-dockerfile-command-introduction.html
- https://www.jianshu.com/p/cbce69c7a52f（好文）
- https://www.cnblogs.com/ityouknow/p/8599093.html
- https://www.jianshu.com/p/efd70ad53602
