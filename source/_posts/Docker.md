---
title: Docker
date: 2021-10-20 16:13:54.023
updated: 2021-10-22 21:04:39.579
url: /archives/docker
categories: 
- 工具
tags: 
---

# 安装docker
以 CentOS7 为例
## 1. 简化安装方法
Docker 官方为了简化安装流程，提供了一套便捷的安装脚本，执行这个脚本后就会自动地将一切准备工作做好，并且把 Docker 的稳定版本安装在系统中。
```shell
curl -fsSL get.docker.com -o get-docker.sh
```
```shell
sh get-docker.sh --mirror Aliyun
```
启动docker
```shell
systemctl start docker
```
## 2. 一般安装
```shell
# 卸载旧的版本
$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```
```shell
# 安装基本的安装包
$ sudo yum install -y yum-utils
```
```shell
# 设置镜像的仓库
$ sudo yum-config-manager \
    --add-repo \
    https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo # 阿里云镜像
```
```shell
# 安装docker引擎
yum install docker-ce docker-ce-cli containerd.io # docker-ce 社区版 ee 企业版
```
```shell
# 启动docker
systemctl start docker
```

# docker命令
## 镜像命令
### 1. docker images
查看 Docker 中当前拥有哪些镜像
```shell
[root@izrz ~]# docker images
REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
MySQL         5.7.32    f07dfa83b528   11 days ago     448MB
tomcat        latest    feba8d001e3f   2 weeks ago     649MB
nginx         latest    ae2feff98a0c   2 weeks ago     133MB
hello-world   latest    bf756fb1ae65   12 months ago   13.3kB
```

### 2. docker search
搜索仓库中的镜像
```shell
docker search MySQL
```

### 3. docker pull
下载镜像，不写版本标志直接执行，则会下载镜像的最新版本
```shell
docker pull MySQL:5.7
```
### 4. docker image rm
删除镜像，若是不指定版本，则默认删除的也是最新版本，还可以通过指定镜像 id 进行删除
```shell
docker image rm MySQL:5.5

docker image rm bf756fb1ae65
```

简化版本`docker rmi 镜像名:版本标志`
删除所有的 MySQL 镜像
```shell
docker rmi -f $(docker images MySQL -q)
```
`docker images MySQL -q`查询出 MySQL 的所有镜像 id，-q表示仅查询 id，并将这些 id 作为参数传递给docker rmi -f指令，这样所有的 MySQL 镜像就都被删除了

## 容器命令
### 1. docker run
通过镜像运行一个容器
```shell
docker run -p 8088:8080 --name tomcat01 tomcat:8.0-jre8
```
-p进行端口映射，第一个 8088 为宿主机端口，第二个 8080 为容器内的端口，外部访问 8088 端口就会通过映射访问容器内的 8080 端口
--name给容器起一个名字
### 2. docker ps
```shell
docker ps -a
```
将运行和非运行的容器全部列举出来

### 3. docker rm
```shell
docker rm d5b6c177c151
```
根据id删除容器

### 4. 启动和停止容器
```shell
docker start
docker restart
docker stop
docker kill
```

### 5. 容器内部
进入
```shell
docker exec -it 容器id /bin/bash
```
退出
```shell
exit
```

# 数据卷
Docker 中的数据卷能够实现宿主机与容器之间的文件共享，它的好处在于对宿主机的文件进行修改将直接影响容器，而无需再将宿主机的文件再复制到容器中。本质上是一个目录挂载，将容器内的目录挂载到虚拟机上。
```shell
docker run -d -p 8080:8080 --name tomcat01 -v /opt/apps:/usr/local/tomcat/webapps tomcat:8.0-jre8
```