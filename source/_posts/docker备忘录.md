---
title: Docker备忘录
date: 2023-03-22 14:29:48
tags: 
    - 备忘录
    - Docker
categories: 备忘录
keywords: Docker
description: Docker 是一种常用的容器化平台，它可以帮助开发人员将应用程序及其依赖项打包为轻量级、可移植的容器。这是一份 Docker 命令备忘录，以便参考和使用。
top_img: https://pic.imgdb.cn/item/647c1399f024cca1732733c8.jpg
cover: https://pic.imgdb.cn/item/647c1399f024cca1732733c8.jpg
---

Docker 是一种常用的容器化平台，它可以帮助开发人员将应用程序及其依赖项打包为轻量级、可移植的容器。下面是一份 Docker 命令备忘录，以便参考和使用。

## 容器生命周期管理

### 运行容器

``` docker
docker run [options] <image_name> [command]
```

其中`options`包括：

``` docker
-d 后台运行容器
-p host_port:container_port 端口映射
-v host_dir:container_dir 目录挂载
--name container_name 为容器指定名称
--restart=always 容器异常退出时自动重启
```

例如，运行名为`my_container`的容器：

``` docker
docker run -d --name <my_container> -p 8080:80 nginx
```

### 列出容器

``` docker
docker ps [options]
```

其中`options`包括：

``` docker
-a 显示所有容器
-q 只显示容器 ID
```

例如，列出所有容器：

``` docker
docker ps -a
```

### 停止容器

``` docker
docker stop <container_name/container_id>
```

例如，停止名为`my_container`的容器：

``` docker
docker stop <my_container>
```

### 删除容器

``` docker
docker rm <container_name/container_id>
```

例如，删除名为 `my_container` 的容器：

``` docker
docker rm <my_container>
```

## 镜像管理

### 获取镜像

``` docker
docker pull <image_name:[tag]>
```

例如，获取最新版本的 nginx 镜像：

``` docker
docker pull nginx
```

### 查看镜像

``` docker
docker images [options] <repository:[tag]>
```

其中 `options` 包括：

`-a` 显示所有镜像

`--digests` 显示镜像的摘要信息

`--no-trunc` 不缩短输出

例如，列出所有镜像：

``` docker
docker images -a
```

## 删除镜像

``` docker
docker rmi <image_name:[tag]>
```

例如，删除名为`my_image`的镜像：

``` docker
docker rmi <my_image>
```

## 日志和调试

### 查看容器日志

``` docker
docker logs <container_name/container_id>
```

例如，查看名为`my_container`的容器的日志：

``` docker
docker logs <my_container>
```

### 进入运行中的容器

``` docker
docker exec -it <container_name/container_id> [command]
```

例如，进入名为`my_container` 的容器的交互式 shell：

``` docker
docker exec -it <my_container> sh
```
