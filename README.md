Docker入门教程
---

![](logo.png)

Docker 是一个开源的应用容器引擎，而一个<ruby>容器<rt>containers</rt></ruby>其实是一个虚拟化的独立的环境，让开发者可以打包他们的应用以及依赖包到一个可移植的容器中，然后发布到任何流行的 Linux 机器上，也可以实现虚拟化。容器是完全使用沙箱机制，相互之间不会有任何接口。

- Docker 的局限性之一，它只能用在 64 位的操作系统上。

目录
===
<!-- TOC -->

- [安装](#安装)
- [命令介绍](#命令介绍)
- [服务管理](#服务管理)
- [镜像管理](#镜像管理)
- [容器管理](#容器管理)
- [使用Docker实战](#使用docker实战)
  - [部署一个Nginx服务](#部署一个nginx服务)
- [Docker私有仓库搭建](#docker私有仓库搭建)
- [参考资料](#参考资料)

<!-- /TOC -->

## 安装

```bash
yum install docker        # CentOS 中安装
apt-get install docker-ce # Ubuntu 中安装
pacman -S docker          # Arch 中安装
emerge --ask docker       # Gentoo 中安装

#=====================

docker version      # 通过查看版本，检查安装是否成功
# Client:
#  Version:         1.12.6
#  API version:     1.24
#  Package version: docker-1.12.6-55.gitc4618fb.el7.centos.x86_64
#  Go version:      go1.8.3
#  Git commit:      c4618fb/1.12.6
#  Built:           Thu Sep 21 22:33:52 2017
#  OS/Arch:         linux/amd64
# 
# Server:
#  Version:         1.12.6
#  API version:     1.24
#  Package version: docker-1.12.6-55.gitc4618fb.el7.centos.x86_64
#  Go version:      go1.8.3
#  Git commit:      c4618fb/1.12.6
#  Built:           Thu Sep 21 22:33:52 2017
#  OS/Arch:         linux/amd64
```

## 命令介绍

```bash
$ docker --help

管理命令:
  container   管理容器
  image       管理镜像
  network     管理网络
命令：
  attach      介入到一个正在运行的容器
  build       根据 Dockerfile 构建一个镜像
  commit      根据容器的更改创建一个新的镜像
  cp          在本地文件系统与容器中复制 文件/文件夹
  create      创建一个新容器
  exec        在容器中执行一条命令
  images      列出镜像
  kill        杀死一个或多个正在运行的容器    
  logs        取得容器的日志
  pause       暂停一个或多个容器的所有进程
  ps          列出所有容器
  pull        拉取一个镜像或仓库到 registry
  push        推送一个镜像或仓库到 registry
  rename      重命名一个容器
  restart     重新启动一个或多个容器
  rm          删除一个或多个容器
  rmi         删除一个或多个镜像
  run         在一个新的容器中执行一条命令
  search      在 Docker Hub 中搜索镜像
  start       启动一个或多个已经停止运行的容器
  stats       显示一个容器的实时资源占用
  stop        停止一个或多个正在运行的容器
  tag         为镜像创建一个新的标签
  top         显示一个容器内的所有进程
  unpause     恢复一个或多个容器内所有被暂停的进程
```

## 服务管理

```bash
service docker start       # 启动 docker 服务，守护进程
service docker stop        # 停止 docker 服务
chkconfig docker on        # 设置为开机启动
```

## 镜像管理

镜像可以看做我们平时装系统的镜像，里面就是一个运行环境。

```bash
docker pull centos:latest  # 从docker.io中下载centos镜像到本地
docker images                       # 查看已下载的镜像
docker rm image_id                 # 删除镜像，指定镜像id
docker rmi $(docker images | grep none | awk '{print $3}' | sort -r)  # 删除所有镜像
```

## 容器管理

容器就像一个类的实例

```bash
docker run centos echo "hello world"  # 在docker容器中运行hello world!
docker run centos yum install -y wget # 在docker容器中，安装wget软件
docker ps                           # 列出包括未运行的容器
docker ps -a                        # 查看所有容器(包括正在运行和已停止的)
docker logs my-nginx                # 查看 my-nginx 容器日志

docker restart my-nginx             # 【重启】容器
docker stop my-nginx                # 【停止运行】一个容器
docker start my-nginx               # 【启动】一个已经存在的容器
docker kill my-nginx                # 【杀死】一个运行中的容器
docker rename my-nginx new-nginx    # 【重命名】容器
docker rm new-nginx                 # 【删除】容器

docker run -i -t centos /bin/bash   # 启动一个容器
docker inspect centos     # 检查运行中的镜像
docker commit 8bd centos  # 保存对容器的修改
docker commit -m "nothing changed" my-nginx my-nginx-image # 使用已经存在的容器创建一个镜像
```

## 使用Docker实战

### 部署一个Nginx服务

1.在 docker hub 中查找 nginx 相关镜像。

```bash
$ docker search nginx

# INDEX       NAME                                                             DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
# docker.io   docker.io/nginx                                                  Official build of Nginx.                        7006      [OK]
# docker.io   docker.io/jwilder/nginx-proxy                                    Automated Nginx reverse proxy for docker c...   1137                 [OK]
# docker.io   docker.io/richarvey/nginx-php-fpm                                Container running Nginx + PHP-FPM capable ...   453                  [OK]
# docker.io   docker.io/jrcs/letsencrypt-nginx-proxy-companion                 LetsEncrypt container to use with nginx as...   230                  [OK]
# docker.io   docker.io/kong                                                   Open-source Microservice & API Management ...   116       [OK]
# docker.io   docker.io/webdevops/php-nginx                                    Nginx with PHP-FPM                              90                   [OK]
```

2.拉取官方镜像，其中上面的非官方镜像是用户们根据自己的需要制作的镜像，方便大家的使用。

```bash
$ docker pull nginx
# Using default tag: latest
# Trying to pull repository docker.io/library/nginx ...
# latest: Pulling from docker.io/library/nginx
# bc95e04b23c0: Pull complete
# 110767c6efff: Pull complete
# f081e0c4df75: Pull complete
# Digest: sha256:004ac1d5e791e705f12a1
```

3.利用这个镜像启动一个新的容器

```bash
docker run --name my-nginx -d -p 8080:80 nginx
# faaed6a2d63af248961aab59713e515c76aea447
```

4.查看容器运行日志

```
docker logs my-nginx
```

启动一个更复杂Nginx的例子：

```bash
# 上面的命令将本地文件中的 nginx.conf 配置文件挂载到容器，并且将要展示的静态页面也挂载到容器。
docker run --name my-nginx \ 
    -v /host/path/nginx.conf:/etc/nginx/nginx.conf:ro \
    -v /some/html:/usr/share/nginx/html:ro \
    -p 8080:80 \
    -d nginx
```

`-v` 参数语法为 `-v host dir:container dir[:ro|rw]`

- --name 为容器取一个名字
- -p 参数语法为 `-p host port:container port`; -p 8080:80 将主机上的8080端口绑定到容器上的80端口，因此在主机中访问8080端口时其实就是访问 nginx 容器的80端口
- -d 后台运行容器

## Docker私有仓库搭建

## 参考资料

- [Docker 快速手册！](https://github.com/eon01/DockerCheatSheet)