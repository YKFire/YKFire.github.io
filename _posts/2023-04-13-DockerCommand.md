---
title: Docker Command
date: 2023-04-13 17:03:00 +0800
categories: [实用工具]
tags: [学习]
pin: true
author: YKFire

toc: true
comments: true

math: false
mermaid: true

typora-root-url: ../../YKFire.github.io

---

# Docker Command

> 最近在整理自己开发过的项目，同时也把项目都部署上线下。把好久没用的docker复习了遍，于是想写篇博客记录下部署过程中常用的命令文件以及需要了解的定义，以免时间一长又忘了~ 

![image-20230413181337939](/assets/blog_res/2023-04-13-DockerCommand.assets/image-20230413181337939.png)

## 一、常用命令

### 镜像操作

- docker images：查看镜像
- docker rmi：删除镜像
- docker pull：从服务器拉取镜像
- docker push：推送镜像到服务器
- docker save：保存镜像为一个压缩包
- docker load：加载压缩包为镜像
- docker build：构建镜像

![image-20230413181203855](/assets/blog_res/2023-04-13-DockerCommand.assets/image-20230413181203855.png)

### 容器操作

> 容器的三个状态
>
> 运行：进程正常运行
>
> 暂停：进程暂停，CPU不再运行，并不释放内存
>
> 停止：进程终止，回收进程占用的内存、CPU等资源-

- docker run：创建并运行一个容器，处于运行状态
- docker pause：让一个运行的容器暂停
- docker unpause：让一个容器从暂停状态恢复运行
- docker stop：停止一个运行的容器
- docker start：让一个停止的容器再次运行
- docker rm：删除一个容器

![image-20230413181225871](/assets/blog_res/2023-04-13-DockerCommand.assets/image-20230413181225871.png)

### docker compose操作

- docker-compose up：执行docker-compose.yml文件 前台启动 (-d 后台启动)
- docker-compose logs：输出日志
- docker-compose ps：列出工程中正在运行的容器 (-a 所有容器）
- docker-compose exec nginx bash：进入工程中指定服务的容器
- docker-compose restart：重启工程中所有服务的容器
- **docker-compose start**：启动工程中所有服务的容器
- **docker-compose stop**：停止工程中所有服务的容器
- docker-compose rm：删除工程中所有服务的容器
- docker-compose images：打印所有服务的容器所对应的镜像



## 二、Dockerfile

​	Dockerfile 是一个文本文件，其内包含了一条条的指令(Instruction)，用于构建镜像。每一条指令构建一层镜像，因此每一条指令的内容，就是描述该层镜像应当如何构建

​	一般搭配docker build命令使用

### 后端模板

```dockerfile
FROM maven:3.5-jdk-8-alpine as builder

# Copy local code to the container image.
WORKDIR /app

COPY ./yupao-backend-0.0.1-SNAPSHOT.jar ./yupao-backend-0.0.1-SNAPSHOT.jar

EXPOSE 8080

# Run the web service on container startup.
CMD ["java","-jar","/app/target/yupao-backend-0.0.1-SNAPSHOT.jar","--spring.profiles.active=prod"]
```



### 前端模板

> 前端编写Dockerfile文件，还需要nginx配置文件 进行代理

```dockerfile
FROM nginx

WORKDIR /usr/share/nginx/html/
USER root

COPY ./nginx.conf /etc/nginx/conf.d/default.conf

COPY ./dist  /usr/share/nginx/html/

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

```tex
server {
    listen 80;

    # gzip config
    gzip on;
    gzip_min_length 1k;
    gzip_comp_level 9;
    gzip_types text/plain text/css text/javascript application/json application/javascript application/x-javascript application/xml;
    gzip_vary on;
    gzip_disable "MSIE [1-6]\.";

    root /usr/share/nginx/html;
    include /etc/nginx/mime.types;

    location / {
        try_files $uri /index.html;
    }

}
```

### 构建镜像

> t：打上版本好  Dockerfile文件应该位于当前命令执行的命令下

```bash
docker build -t 项目名:v0.0.1 .
```

### 创建容器

> 前面为宿主端口 后面为容器内端口

```bash
docker run -p 80:80 -d xxx:v0.0.1

docker run -p 8081:8080 -d xxx:v0.0.1
```



## 三、docker-compose.yml

​	Docker Compose是一个用于定义和运行多容器应用程序的工具。 通过compose，我们可以使用yaml文件来配置应用程序的服务，然后使用一个命令来创建和启动所有已配置的服务。	

​	**主要用于一键部署所有的容器**，当你的项目存在多个容器时，使用docker compose可以将创建每个容器并执行的命令整合起来，以及定义网络和数据集的挂载，甚至连构建镜像的过程也可以一起包含，可以简化大量重复的手动工作。

> ​	docker-compose.yml文件是一个定义服务、 网络和卷的 YAML 文件 。Compose 文件的默认路径是 ./docker-compose.yml

### 模板

```yml
version: '3.9'
services:
  web-ui:
    image: partner-dating-ui:v0.0.1
    restart: always
    container_name: part-ui
    ports:
      - "80:80"
  web:
    image: partner-dating:v0.0.2
    restart: always
    container_name: part
    ports:
      - "8081:8080"
```

> 更多关于docker-compose.yml文件的解析与使用：https://blog.csdn.net/qq_37768368/article/details/120583141

![image-20230413181128653](/assets/blog_res/2023-04-13-DockerCommand.assets/image-20230413181128653.png)

