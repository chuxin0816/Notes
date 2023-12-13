# Docker

[TOC]

## 常用命令

### 帮助命令

```shell
systemctl start docker #启动docker服务
docker version #版本信息
docker info    #系统信息，包括镜像和容器数目
docker --help  #帮助命令
```

### 镜像命令

```shell
docker images #查看本地镜像
	-a #显示所有镜像
	-q #只显示镜像ID
docker search xx #搜索镜像
docker pull xx   #下载镜像
	:tag #指定版本号
docker rmi -f #强制删除镜像
```

### 容器命令

> 有了镜像才可以创建容器

```shell
docker run [可选参数] 镜像名 #新建容器并启动 
	--name "name" #指定容器名称
	-d            #后台运行（需组合-it使用）
	-it           #以交互方式运行，进入容器查看内容（bash）
	-p            #指定端口
			主机端口:容器端口(常用)
			容器端口
	-P                       #随机端口（大写）
常用：docker run --name=xx -it --cap-add=SYS_PTRACE --security-opt seccomp=unconfined -p xx:22 -v 本地目录:容器目录 镜像名 bash
#--cap-add=SYS_PTRACE --security-opt seccomp=unconfined 开启调试功能
docker ps                  #查看正在运行的容器
	-a            #查看历史记录 
	-n=?          #显示最近的几行
	-q            #只显示容器ID
exit                       #退出容器（并停止）
control+P,control+Q                #退出容器（不停止）
docker start 容器ID         #启动容器
docker restart 容器ID       #重启容器
docker stop 容器ID          #停止容器
docker rm 容器ID            #删除容器
docker kill 容器ID          #强制删除
```

### 其他命令

```shell
docker logs 容器ID     #查看日志
	-tf                 #显示具体信息
	--tail n            #显示n行
docker inspect 容器ID  #查看容器具体信息（如挂载信息）
docker top 容器ID      #查看进程信息
docker exec -it 容器ID bash #进入后台运行的容器（开启新的终端）
docker attach 容器ID   #进入后台运行的容器（进入正在执行的终端）
docker cp 容器ID:路径 字节路径 #从容器拷贝文件到主机
docker cp 字节路径 容器ID:路径 #从主机拷贝文件到容器
docker stats          #查看CPU状态
docker image prune    #删除没有tag的镜像
```

## 提交镜像

> 保存当前容器状态，获得镜像

```shell
docker commit -m="提交的描述信息" -a="作者" 容器ID 目标镜像名:[TAG]
```

## 上传镜像到Dockhub

```shell
docker tag local-image:tagname new-repo:tagname
docker push new-repo:tagname
```

## 容器数据卷

> 将文件同步到本地

```shell
docker run -it -v 本地目录:容器目录[:ro/rw] 镜像名 bash
#ro->readonly		rw->readwrite(默认)
#ro只可以通过宿主机操作，容器内无法操作
```

### 匿名和具名挂载

> 匿名挂载一般在/var/lib/docker/volumes/xx/_data

```shell
docker run -v 容器路径 镜像名     #匿名挂载
docker run -v 卷名:容器路径 镜像名 #具名挂载
docker volume ls                #查看所有卷
docker volume inspect 卷名       #查看卷的详细信息
```

### 数据卷容器（继承）

> 多个容器同步数据（继承）

```shell
docker run --name 容器名 --volumes-from 父容器名 镜像名
#同步父容器的挂载信息
```

##  Dockerfile

> 用于构建镜像

### 指令

```dockerfile
FROM <image>:<tag>       #Ubuntu，CentOS等
LABEL <name>     #作者信息
RUN <command>          #在镜像中执行命令
COPY <src> <dest>     #将本地文件复制到镜像中
ADD <src> <dest>     #将本地文件或URL复制到镜像中
WORKDIR /path/to/workdir#指定工作目录
ENV <key> <value>      #设置环境变量
EXPOSE <port>          #暴露端口
CMD <command>        #指定容器启动命令（可以被docker run覆盖，只有最后一条有效）
ENTRYPOINT <command> #指定容器启动命令（不会被覆盖，只有最后一个会生效）
USER <username>     #指定运行镜像的用户
VOLUME /path/to/volume  #创建挂载点
```

### 构建

1. 编写Dockerfile文件
2. 运行`docker build -t 目标镜像名:TAG .`

`docker history 镜像ID` 查看镜像构建过程

## 构建C++开发环境

```dockerfile
FROM ubuntu:22.04
ENV TZ=Asia/Shanghai
ENV LANG=C.UTF-8
LABEL maintainer=chuxin<2670957513@qq.com>
WORKDIR /root
RUN apt-get update && apt-get install -y \
    git \
    cmake \
    build-essential \
    clangd \
    lldb \
    vim \
    && rm -rf /var/lib/apt/lists/*
CMD ["./root"]
```

```shell
docker run --name=xx -it --cap-add=SYS_PTRACE --security-opt seccomp=unconfined -p xxx:22 -v 本地目录:容器目录 镜像名:TAG bash
```

