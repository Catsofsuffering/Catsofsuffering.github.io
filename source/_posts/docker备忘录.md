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
## Docker安装

```bash
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  
sudo apt-get update

sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

## Docker代理
### 配置 Docker Daemon 使用代理 (Docker pull生效) 

创建或编辑 `/etc/systemd/system/docker.service.d/http-proxy.conf` 文件：

```ini
[Service]
Environment="HTTP_PROXY=http://proxy_ip:proxy_port"
Environment="HTTPS_PROXY=http://proxy_ip:proxy_port"
Environment="NO_PROXY=localhost,127.0.0.1,docker-registry.example.com"
```
   
重启 Docker 使配置生效：
   
```bash
sudo systemctl daemon-reload
sudo systemctl restart docker
```

### 配置容器内的代理（Docker run生效）

**临时方法**：启动容器时，可以通过环境变量传递代理配置：

```bash
docker run -e HTTP_PROXY=http://proxy_ip:proxy_port -e HTTPS_PROXY=http://proxy_ip:proxy_port my_image
```

在容器运行阶段，如果需要代理上网，则需要配置 ~/.docker/config.json。以下配置，只在Docker 17.07及以上版本生效。

```text
{
 "proxies":
 {
   "default":
   {
     "httpProxy": "http://proxy_ip:proxy_port",
     "httpsProxy": "http://proxy_ip:proxy_port",
     "noProxy": "localhost,127.0.0.1,.example.com"
   }
 }
}
```

这个是用户级的配置，除了 proxies，docker login 等相关信息也会在其中。而且还可以配置信息展示的格式、插件参数等。

此外，容器的网络代理，也可以直接在其运行时通过 -e 注入 http_proxy 等环境变量。这两种方法分别适合不同场景。config.json 非常方便，默认在所有配置修改后启动的容器生效，适合个人开发环境。在CI/CD的自动构建环境、或者实际上线运行的环境中，这种方法就不太合适，用 -e 注入这种显式配置会更好，减轻对构建、部署环境的依赖。当然，在这些环境中，最好用良好的设计避免配置代理上网。
### Docker Build 代理

虽然 docker build 的本质，也是启动一个容器，但是环境会略有不同，用户级配置无效。在构建时，需要注入 http_proxy 等参数。

```text
docker build . \
    --build-arg "HTTP_PROXY=http://proxy.example.com:8080/" \
    --build-arg "HTTPS_PROXY=http://proxy.example.com:8080/" \
    --build-arg "NO_PROXY=localhost,127.0.0.1,.example.com" \
    -t your/image:tag
```

**注意**：无论是 docker run 还是 docker build，默认是网络隔绝的。如果代理使用的是 localhost:3128 这类，则会无效。这类仅限本地的代理，必须加上 --network host 才能正常使用。而一般则需要配置代理的外部IP，而且代理本身要开启 Gateway 模式。

## 将 Docker 作为非 root 用户管理

Docker 守护进程绑定到 Unix 套接字，而不是 TCP 端口。默认情况下，Unix 套接字的所有者是 `root` 用户，其他用户只能通过 `sudo` 来访问它。Docker 守护进程始终以 `root` 用户身份运行。

如果不希望每次在运行 `docker` 命令前都加上 `sudo`，可以创建一个名为 `docker` 的 Unix 用户组，并将用户添加到这个组中。当 Docker 守护进程启动时，它会创建一个可供 `docker` 组成员访问的 Unix 套接字。在某些 Linux 发行版中，使用包管理器安装 Docker 引擎时，系统会自动创建这个组。在这种情况下，你无需手动创建该组。

1. 创建 `docker` 组：

```bash
sudo groupadd docker
sudo usermod -aG docker $USER
# test
docker run hello-world
```
### Trouble Shooting

如果在将用户添加到 `docker` 组之前，使用 `sudo` 运行过 Docker CLI 命令，可能会看到以下错误：

```
WARNING: Error loading config file: /home/user/.docker/config.json -
stat /home/user/.docker/config.json: permission denied
```

这个错误表示 `~/.docker/` 目录的权限设置不正确，这是由于之前使用了 `sudo` 命令。

为了解决这个问题，可以选择删除 `~/.docker/` 目录（该目录会自动重新创建，但任何自定义设置都会丢失），或者使用以下命令更改其所有权和权限：

```bash
sudo chown "$USER":"$USER" /home/"$USER"/.docker -R
sudo chmod g+rwx "$HOME/.docker" -R
``` 

这会确保 `~/.docker/` 目录的权限设置正确，允许在不使用 `sudo` 的情况下运行 Docker 命令。
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
