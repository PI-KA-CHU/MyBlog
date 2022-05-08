+++
author = "pikachu"
title = "Docker实战：1、安装及常见命令"
date = "2019-08-17"
description = " "
tags = [
	"docker",
	"部署工具"
]
categories = [
    "it", "开发运维"
]

+++



## 一、Docker安装（Centos7）

- 1、Docker要求Centos内核要高于3.10，所以首先应该检查是否满足要求（如果是本地虚拟机，需要保证可以联网，可以通过`ping`检测）
```
uname -r
```
- 2、使用 root 权限登录 Centos。确保 `yum` 包更新到最新。
```
sudo yum update
```
- 3、卸载旧版本(如果安装过旧版本的话)
```
sudo yum remove docker  docker-common docker-selinux docker-engine
```
- 4、安装需要的软件包， `yum-util` 提供`yum-config-manager`功能，另外两个是`devicemapper`驱动依赖的
```
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```
- 5、设置yum源
```
# 阿里云版（推荐）
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

# 官方版（国内速度慢）
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```
- 6、可以查看所有仓库中所有docker版本，并选择特定版本安装
```
yum list docker-ce --showduplicates | sort -r
```
![image](https://user-images.githubusercontent.com/38284818/62308234-a1518580-b4b7-11e9-9373-96c8f5d10fac.png)

- 7、更新 yum 软件源缓存，并安装 docker
```
# 更新 yum 软件源缓存
sudo yum makecache fast

# 默认安装最新版本
sudo yum install docker-ce
# 指定版本下载，例如：sudo yum install docker-ce-17.12.0.ce
sudo yum install <FQPN>
```
- 8、启动并加入开机自启
```
sudo systemctl start docker
sudo systemctl enable docker
```
- 9、验证安装是否成功
```
docker version
```
![image](https://user-images.githubusercontent.com/38284818/62308781-b24ec680-b4b8-11e9-9f05-2836cf7d09d7.png)

- 10、Docker镜像加速器

添加镜像加上文件
```
# 在 /etc/docker/daemon.json 中写入如下内容（如果文件不存在请新建该文件）
{
  "registry-mirrors": [
    "https://docker.mirrors.ustc.edu.cn"
  ]
}
```

重新服务
```
sudo systemctl restart docker
```

&nbsp;
&nbsp;

## 二、Docker常见命令（以Redis为例）

- 镜像拉取
```
docker pull redis:latest
```
![image](https://user-images.githubusercontent.com/38284818/62311135-b5988100-b4bd-11e9-87a3-51cdfa3c552e.png)

- 获取本地镜像列表
```
docker image ls / docker images
```
![image](https://user-images.githubusercontent.com/38284818/62311308-2344ad00-b4be-11e9-8e8d-abbb10aa00c6.png)

- 运行镜像
```
docker run 镜像名称 &   # 注: &表示后台运行
```

- 删除本地镜像
```
docker image rm <镜像>  # 注：<镜像>可以是 镜像短 ID、镜像长 ID、镜像名 或者 镜像摘要
```
> 问题：删除的时候可能会报镜像被容器引用的异常：
> Error response from daemon: conflict: unable to remove repository reference "hello-world" (must > > force) - container d4f4b4789f90 is using its referenced image fce289e99eb9
> 
> 解决：输入命令 `docker ps -a` 显示当前所有容器（包括未运行的），然后使用 `docker rm 容器ID` > 将其删除后即可正常删除镜像。

- 查看容器
```
docker ps -a   # 注：-a表示查看所有容器，包括未运行的，不加只显示当前运行的容器
```
- 容器启动、重启及停止
```
docker start container_name/container_id
docker stop container_name/container_id
docker restart container_name/container_id
```
- 删除容器
```
docker rm container_name/container_id
```
- 删除所有停止的容器
```
docker rm $(docker ps -a -q)
```
- 查看docker信息
```
docker info
```
- 查找Docker Hub上的nginx镜像
```
docker search nginx
```

&nbsp;

### 

## 三、参考

- https://www.cnblogs.com/yufeng218/p/8370670.html
- https://www.funtl.com/zh/docs-docker/CentOS-%E5%AE%89%E8%A3%85-Docker.html#%E4%BD%BF%E7%94%A8-yum-%E5%AE%89%E8%A3%85
- 镜像加速： https://blog.csdn.net/qq_37495786/article/details/83246421
- redis镜像： https://blog.csdn.net/cookily_liangzai/article/details/80726163
- Docker命令： https://snailclimb.top/JavaGuide/#/tools/Docker-Image?id=%E4%B8%80-docker-%E4%B8%8B%E8%BD%BD%E9%95%9C%E5%83%8F
- http://www.ityouknow.com/docker/2018/03/07/docker-introduction.html