---
title: Docker容器-基础篇
tags:
  - Docker
  - 容器
categories: Linux
top: 1
cover: 'https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20211212170545.png'
abbrlink: e33c148b
date: 2021-12-12 16:25:44
updated: 2022-06-08 18:56:24
---

# Docker 容器-基础篇

## Docker 概述

### 1. Docker 为什么会出现？

> 环境切换/配置麻烦

一般一个产品的生命周期中，可能会存在多个环境：

- 开发环境
- 测试环境
- 生产环境

其实我们在编程的过程中，很大一部分时间都花在 `环境` 上：

- 比如重装系统之后，想要运行 `Jar/War` 包，就必须在系统里装上 `JDK` . `Tomcat`. `MySQL` 等环境 ，并配置好相应的环境变量
- 以前生产环境和测试环境完全是两套不同的环境，可能会出现：==代码在测试环境跑没问题，到生产环境就出各种错！==
- 在学习 `分布式/集群` 项目时，需要搭建多个环境，以前使用 `Vmware` 搭建费时费力，且对电脑的配置要求较高

> 应用之间需要隔离

- 假设，我只有一台服务器，我写了两个应用（网站），都部署在一台服务器里，倘若其中一个应用出现了问题，导致 CPU 跑满到 100%，那么另一个应用也会受影响！
- 同一个服务器下端口冲突. JRE 版本冲突...

Docker 的出现就为以上问题带来了解决方案：

Docerk 的思想就来自于集装箱！

![](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20211212012101.png)

### 2. Docker 是什么？

#### Docker 基本介绍

`Docker` 是一个开源的应用容器引擎，基于 `Go 语言` 并遵从 `Apache2.0` 协议开源。

`Docker` 可以让开发者打包他们的应用以及依赖包到一个轻量级. 可移植的容器中，然后发布到任何流行的 Linux 机器上，也可以实现虚拟化。

容器是完全使用沙箱机制，相互之间不会有任何接口（类似 iPhone 的 app），更重要的是容器性能开销极低。

`Docker` 从 `17.03` 版本之后分为 `CE（Community Edition: 社区版）` 和 `EE（Enterprise Edition: 企业版）`，我们用社区版就可以了。

官方文档：https://docs.docker.com/

#### 应用场景

- Web 应用的自动化打包和发布。
- 自动化测试和持续集成. 发布。
- 在服务型环境中部署和调整数据库或其他的后台应用。
- 从头编译或者扩展现有的 `OpenShift` 或 `Cloud Foundry` 平台来搭建自己的 `PaaS` 环境。

#### Docker 的优势

- 更快速地进行应用的交付和部署
- 更便携的升级和扩容
- 更简单的系统运维
- 更高效的计算机资源利用

> 总结：解决了==运行环境和配置问题==的==软件容器==，方便做持续集成并有助于整体发布的容器虚拟化技术。

### 3. 虚拟化技术和容器化技术的区别

- 虚拟化技术：
  1. 资源占用多
  1. 冗余步骤多
  1. 启动很慢


- 容器化技术：容器化技术不是模拟的一个完整的操作系统

比较 Docker 和虚拟机的不同：

1. 传统虚拟机，虚拟出硬件，运行一个完整的操作系统，然后在这个系统上安装和运行软件。
2. Docker 容器内的应用直接运行在宿主机的内容，容器是没有自己的内核的，也没有虚拟硬件。
3. 每个容器都是相互隔离的，每个容器都有属于自己的文件系统，互不影响。

### 4. Docker 的基本组成

Docker 的基本组成图如下：

![image-20211212134734666](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20211212134734.png)

- **镜像（image）**

  ```
  Docker 镜像（Image）就是一个只读的模板。镜像可以用来创建 Docker 容器，一个镜像可以创建很多容器。就好似 Java 中的类和对象，类就是镜像，容器就是对象！
  ```

- **容器（container）**

  ```
  Docker 利用容器（Container）独立运行的一个或一组应用。容器就用镜像创建的运行实例。
  
  它可以被启动. 开始. 停止. 删除。每个容器都是相互隔离的，保证安全的平台。
  
  可以把容器看作是一个简易版的 Linux 环境（包括root用户权限. 进程空间. 用户空间和网络空间）和运行在其中的应用程序。
  
  容器的定义和镜像几乎一模一样，也是一堆层的统一视角，唯一区别在于容器的最上面那层是可读可写的。
  ```

* **仓库（repository）**

  ```
  仓库（Repository）是集中存放镜像文件的场所。
  
  仓库（Repository）和仓库注册服务器（Registry）是有区别的。仓库注册服务器上往往存放着多个仓库，每个仓库中又包含了多个镜像，每个镜像有不同的标签（tag）。
  
  仓库分为公开仓库（Public）和私有仓库（Private）两种形式。
  
  最大的公开仓库是 Docker Hub（https://hub.docker.com/），存放了数量庞大的镜像供用户下载。
  国内最大的公开仓库包括阿里云. 网易云等。
  ```

## Docker 安装

### 1. Docker 的安装与卸载

> 环境准备

1. 需要会一点点的 Linux 的基础
2. CentOS 7
3. 使用 Xshell 连接远程服务器进行操作！

> 环境查看

```shell
# 系统内核是 3.10 以上的
[root@ouwen666 ~]# uname -r
3.10.0-1062.18.1.el7.x86_64
# 系统版本 centOS7
[root@ouwen666 ~]# cat /etc/os-release
NAME="CentOS Linux"
VERSION="7 (Core)"
ID="centos"
ID_LIKE="rhel fedora"
VERSION_ID="7"
PRETTY_NAME="CentOS Linux 7 (Core)"
ANSI_COLOR="0;31"
CPE_NAME="cpe:/o:centos:centos:7"
HOME_URL="https://www.centos.org/"
BUG_REPORT_URL="https://bugs.centos.org/"

CENTOS_MANTISBT_PROJECT="CentOS-7"
CENTOS_MANTISBT_PROJECT_VERSION="7"
REDHAT_SUPPORT_PRODUCT="centos"
REDHAT_SUPPORT_PRODUCT_VERSION="7"
```

> 安装

帮助文档：[Docker 官方帮助文档](https://docs.docker.com/engine/install/centos/)

```shell
# 1. 卸载旧的版本
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine

# 2. 需要的安装包
yum install -y yum-utils

# 3. 设置镜像的仓库
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo # 默认是国外的，十分慢！
# 建议使用阿里云的镜像地址
yum-config-manager \
    --add-repo \
    https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

# 更新yum软件包索引
yum makecache fast

# 4. 安装docker相关的 docker-ce 社区版 ee 企业版
yum install docker-ce docker-ce-cli containerd.io

# 5. 启动docker
systemctl start docker

# 6. 使用docker version查看是否安装成功
```

![image-20211212144007872](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20211212144008.png)

```shell
# 7. 启动docker-hello-world
docker run hello-world
```

![image-20211212144259750](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20211212144259.png)

```shell
# 8. 查看下载的这个hello-world镜像
[root@ouwen666 /]# docker images
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
hello-world   latest    feb5d9fea6a5   2 months ago   13.3kB
```

> 卸载 Docker

```shell
# 1. 卸载依赖
yum remove docker-ce docker-ce-cli containerd.io
# 2. 删除资源 docker的默认工作路径
rm -rf /var/lib/docker
rm -rf /var/lib/containerd
```

### 2. 阿里云镜像加速

- 登录阿里云，找到容器镜像服务

  ![image-20211218143146762](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20211218143348.png)

- 找到镜像加速地址

  ![image-20211218143621304](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20211218143621.png)

- 配置使用

  ```shell
  sudo mkdir -p /etc/docker
  
  sudo tee /etc/docker/daemon.json <<-'EOF'
  {
    "registry-mirrors": ["https://alq7pwwu.mirror.aliyuncs.com"]
  }
  EOF
  
  sudo systemctl daemon-reload
  
  sudo systemctl restart docker
  ```

### 3. 回顾 HelloWorld 流程

![image-20211218144212215](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20211218144212.png)

### 4. 底层原理

#### Docker 是怎么工作的？

Docker 是一个 Client - Server 结构的系统，Docker 的守护进程运行在主机上。通过 Socket 从客户端访问！

DockerServer 接收到 Docker - Client 的指令，就会执行这个命令！

![image-20211218144701740](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20211218144701.png)

#### Docker 为什么比 VM 快？

1. Docker 有着比虚拟机更少的抽象层

2. Docker 利用的是宿主机的内核，VM 需要的是 Guest OS（操作系统）

   ![image-20211218144918851](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20211218144918.png)

   新建一个容器的时候，Docker 不需要像虚拟机一样重新加载一个操作系统内核，避免不必要的消耗。

## Docker 的常用命令

### 1. 帮助命令

```shell
# 显示docker的版本信息
docker version
# 显示docker的系统信息，包括镜像和容器
docker info
# 帮助命令
docker 命令 --help
```

帮助文档地址：[Docker 官方帮助文档](https://docs.docker.com/engine/reference/commandline/cli/)

### 2. 镜像命令

#### 列出本机所有镜像

```shell
[root@ouwen666 ~]# docker images
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
hello-world   latest    feb5d9fea6a5   2 months ago   13.3kB

# 解释
REPOSITORY  镜像的仓库源
TAG         镜像的标签
IMAGE ID    镜像的ID
CREATED     镜像的创建时间
SIZE        镜像的大小

# 可选项
  -a， --all             # 列出所有镜像
  -q， --quiet           # 只显示镜像的ID
```

#### 搜索镜像

```shell
[root@ouwen666 ~]# docker search mysql
NAME                              DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
mysql                             MySQL is a widely used， open-source relation…   11833     [OK]
mariadb                           MariaDB Server is a high performing open sou…   4505      [OK]

# 可选项，通过收藏来过滤
--filter=STARS=3000
[root@ouwen666 ~]# docker search mysql --filter=STARS=3000
NAME      DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
mysql     MySQL is a widely used， open-source relation…   11833     [OK]
mariadb   MariaDB Server is a high performing open sou…   4505      [OK]
```

#### 下载镜像

```shell
# 下载镜像 docker pull 镜像名[:tag]
[root@ouwen666 ~]# docker pull mysql
Using default tag: latest   # 如果写 tag，默认就是 latest
latest: Pulling from library/mysql
ffbb094f4f9e: Pull complete # 分层下载，docker image 的核心 联合文件系统
df186527fc46: Pull complete
fa362a6aa7bd: Pull complete
5af7cb1a200e: Pull complete
949da226cc6d: Pull complete
bce007079ee9: Pull complete
eab9f076e5a3: Pull complete
8a57a7529e8d: Pull complete
b1ccc6ed6fc7: Pull complete
b4af75e64169: Pull complete
3aed6a9cd681: Pull complete
23390142f76f: Pull complete
Digest: sha256:ff9a288d1ecf4397967989b5d1ec269f7d9042a46fc8bc2c3ae35458c1a26727  # 签名
Status: Downloaded newer image for mysql:latest
docker.io/library/mysql:latest  # 真实地址

# 等价与它
docker pull mysql
docker pull docker.io/library/mysql:latest

# 指定版本下载
[root@ouwen666 ~]# docker pull mysql:5.7
5.7: Pulling from library/mysql
ffbb094f4f9e: Already exists
df186527fc46: Already exists
fa362a6aa7bd: Already exists
5af7cb1a200e: Already exists
949da226cc6d: Already exists
bce007079ee9: Already exists
eab9f076e5a3: Already exists
c7b24c3f27af: Pull complete
6fc26ff6705a: Pull complete
bec5cdb5e7f7: Pull complete
6c1cb25f7525: Pull complete
Digest: sha256:d1cc87a3bd5dc07defc837bc9084f748a130606ff41923f46dec1986e0dc828d
Status: Downloaded newer image for mysql:5.7
docker.io/library/mysql:5.7
```

![image-20211218170934412](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20211218170934.png)

#### 删除镜像

```shell
[root@ouwen666 ~]# docker rmi -f 镜像ID  # 删除指定的镜像
[root@ouwen666 ~]# docker rmi -f 镜像ID 镜像ID 镜像ID 镜像ID  # 删除多个镜像
[root@ouwen666 ~]# docker rmi -f $(docker images -aq)   # 删除全部的镜像
```

### 3. 容器命令

有了镜像才可以创建容器：linux，下载一个 centos 镜像来学习

```shell
docker pull centos
```

#### 新建容器并启动

```shell
docker run [可选参数] image

# 参数说明
--name="Name"  # 容器名字 为容器指定一个名称
-d             # 后台方式运行
-it            # 使用交互方式运行，并分配一个伪终端，等待交互
-p             # 指定容器的端口  -p 8080:8080
  -p ip:主机端口:容器端口 (常用)
  -p 主机端口:容器端口 (常用)
  -p 容器端口
-P             # 随机指定端口

# 测试一下 启动并进入容器
[root@ouwen666 ~]# docker run -it centos /bin/bash
# 查看容器内的centos，官方镜像是一个基础版本，很多命令都是不完善的！
[root@0226b99be9ff /]# ls
bin  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
# 从容器中退出主机
[root@0226b99be9ff /]# exit
exit
```

#### 列出所有运行的容器

```shell
# docker ps 命令
-a    # 列出当前正在运行的容器+带出历史运行过的容器
-l    # 显示最近创建的容器
-n=?  # 显示最近创建n个容器
-q    # 只显示容器的编号
[root@ouwen666 ~]# docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
[root@ouwen666 ~]# docker ps -a
CONTAINER ID   IMAGE          COMMAND       CREATED         STATUS                             NAMES
0226b99be9ff   centos         "/bin/bash"   3 minutes ago   Exited (0) About a minute ago
55a3ece5f682   feb5d9fea6a5   "/hello"      6 days ago      Exited (0) 6 days ago
```

#### 退出容器

```shell
exit    # 直接停止容器并退出
Ctrl + P + Q  # 容器不停止并退出
```

#### 删除容器

```shell
docker rm 容器ID  # 删除指定的容器，不能删除正在运行的容器，如果要强制删除 rm -f
docker rm -f $(docker ps -aq)  # 删除所有的容器
docker ps -a -q|xargs docker rm  #删除所有的容器
```

#### 启动和停止容器

```shell
docker start 容器ID       # 启动容器
docker restart 容器ID     # 重启容器
docker stop 容器ID        # 停止容器
docker kill 容器ID        # 强制停止当前容器
```

### 4. 常用其他命令【重要】

#### 后台启动容器

```shell
# 命令 docker run -d 镜像名！
[root@ouwen666 /]# docker run -d centos

# 问题 docker ps，发现 centos 停止了

# 常见的坑：docker容器使用后台运行，就必须要有要给前台进程，docker发现没有应用，就会自动停止
```

#### 查看日志

```shell
docker logs -f -t --tail 10 容器ID

# 写一段shell脚本，不停的打印输出
[root@ouwen666 /]# docker run -d centos /bin/sh -c "while true;do echo helloworld;sleep 1;done"
5fff272a8948f573264a09ac17d437b6d7424a5b03604b4191666f252993a6f3

[root@ouwen666 /]# docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS
5fff272a8948   centos    "/bin/sh -c 'while t…"   5 seconds ago   Up 5 seconds

# 显示日志
-tf             # 显示所有的日志
--tail number   # 显示指定行数的日志
[root@ouwen666 /]# docker logs -tf --tail 10 5fff272a8948

```

#### 查看容器中的进程信息

```shell
# 命令 docker top 容器ID
[root@ouwen666 /]# docker top 5fff272a8948
UID                 PID                 PPID                C                   STIME
root                326                 32457               0                   17:46
root                32457               32438               0                   17:42
```

#### 查看镜像/容器的详细信息

```shell
# 命令 docker inspect 镜像/容器ID
[root@ouwen666 /]# docker inspect 5fff272a8948
```

#### 进入当前正在运行的容器

```shell
# 通常容器都是使用后台方式运行的，需要进入容器，修改一些配置

# 命令
docker exec -it 容器ID bashShell
# 测试
[root@ouwen666 /]# docker exec -it 5fff272a8948 /bin/bash
[root@5fff272a8948 /]#

# 方式二
docker attach 容器ID
# 测试
[root@ouwen666 /]# docker attach 5fff272a8948
正在执行的代码...

# docker exec     # 进入容器后，开启一个新的终端，exit后不会停止容器（常用）
# docker attach   # 进入容器正在执行的终端，不会启动新的进程，exit后会停止容器！
```

#### 从容器内拷贝文件到主机上

```shell
docker cp 容器ID:容器内路径 目的主机路径
# 测试
[root@ouwen666 home]# docker ps
CONTAINER ID   IMAGE     COMMAND       CREATED              STATUS
7938b1a7dece   centos    "/bin/bash"   About a minute ago   Up About a minute
# 进入容器内部
[root@ouwen666 home]# docker attach 7938b1a7dece
[root@7938b1a7dece /]# cd /home/
# 创建一个文件
[root@7938b1a7dece home]# touch helloworld.java
[root@7938b1a7dece home]# ls
helloworld.java
# 退出容器
[root@7938b1a7dece home]# exit
exit
# 将容器中的文件拷贝到主机中
[root@ouwen666 home]# docker cp 7938b1a7dece:/home/helloworld.java /home/
[root@ouwen666 home]# ls
git  helloworld.java
```

#### 导入/导出容器

- export 导出容器的内容留作为一个tar归档文件[对应import命令]

- import 从tar包中的内容创建一个新的文件系统再导入为镜像[对应export]

```
# 导出
[root@88233 ~]# docker export 71720f3a8f51 > myubuntu.tar

# 导入
[root@88233 ~]# cat myubuntu.tar | docker import - vansys/ubuntu:1.0
```

### 5. 小结

![image-20211218192225807](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20211218192225.png)

```
attach       # 当前 shell 下 attach 连接指定运行镜像
build        # 通过 Dockerfile 定制镜像
commit       # 提交当前容器为新的镜像
cp           #从容器中拷贝指定文件或者目录到宿主机中
create       # 创建一个新的容器，同 run，但不启动容器
diff         # 查看 docker 容器变化
events       # 从 docker 服务获取容器实时事件
exec         # 在已存在的容器上运行命令
export       # 导出容器的内容流作为一个 tar 归档文件[对应 import ]
history      # 展示一个镜像形成历史
images       # 列出系统当前镜像
import       # 从tar包中的内容创建一个新的文件系统映像[对应export]
info         # 显示系统相关信息
inspect      # 查看容器详细信息
kill         # kill 指定 docker 容器
load         # 从一个 tar 包中加载一个镜像[对应 save]
login        # 注册或者登陆一个 docker 源服务器
logout       # 从当前 Docker registry 退出
logs         # 输出当前容器日志信息
port         # 查看映射端口对应的容器内部源端口
pause        # 暂停容器
ps           # 列出容器列表                        
pull         # 从docker镜像源服务器拉取指定镜像或者库镜像
push         # 推送指定镜像或者库镜像至docker源服务器
restart      # 重启运行的容器
rm           # 移除一个或者多个容器
rmi          # 移除一个或多个镜像[无容器使用该镜像才可删除，否则需删除相关容器才可继续或 -f 强制删除]
run          # 创建一个新的容器并运行一个命令
save         # 保存一个镜像为一个 tar 包[对应 load]
search       # 在 docker hub 中搜索镜像
start        # 启动容器
stop         # 停止容器
tag          # 给源中镜像打标签
top          # 查看容器中运行的进程信息
unpause      # 取消暂停容器
version      # 查看 docker 版本号
wait         # 截取容器停止时的退出状态值
```

### 6. 练习

#### Docker 安装 Nginx

```shell
# 1. 搜索镜像 或者去 dockerHub 上搜索 https://hub.docker.com/search?q=nginx&type=image
[root@ouwen666 home]# docker search nginx
# 2. 下载镜像
[root@ouwen666 home]# docker pull nginx
# 3. 运行测试
[root@ouwen666 home]# docker images
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
nginx        latest    f652ca386ed1   2 weeks ago    141MB
centos       latest    5d0da3dc9764   3 months ago   231MB
# -d 后台运行
# --name 给容器命名
# -p 宿主机端口:容器内部端口 映射端口
[root@ouwen666 home]# docker run -d --name nginx01 -p 3344:80 nginx
639cb4f9a9e60d96d698f0c1200f216176a3735b40b3276b25af5e8fb502e337
[root@ouwen666 home]# docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS         PORTS                  NAMES
639cb4f9a9e6   nginx     "/docker-entrypoint.…"   10 seconds ago   Up 9 seconds   0.0.0.0:3344->80/tcp   nginx01
[root@ouwen666 home]# curl localhost:3344

# 进入容器
[root@ouwen666 home]# docker exec -it nginx01 /bin/bash
root@639cb4f9a9e6:/# whereis nginx
nginx: /usr/sbin/nginx /usr/lib/nginx /etc/nginx /usr/share/nginx
root@639cb4f9a9e6:/# cd /etc/nginx/
root@639cb4f9a9e6:/etc/nginx# ls
conf.d	fastcgi_params	mime.types  modules  nginx.conf  scgi_params  uwsgi_params
root@639cb4f9a9e6:/etc/nginx#
```

> 端口暴露：

![image-20211218194409553](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20211218194409.png)

#### Docker 安装 Tomcat

```shell
# 官方的使用
docker run -it --rm tomcat:9.0
# 这种方式停止了容器之后，会直接删除容器

# 下载再启动
docker pull tomcat:9.0

# 启动运行
docker run -d -p 3355:8080 --name tomcat01 tomcat:9.0

# 测试访问没有问题

# 进入容器内部
[root@ouwen666 home]# docker exec -it tomcat01 /bin/bash
root@b59126dcef8d:/usr/local/tomcat# ls
BUILDING.txt	 LICENSE  README.md	 RUNNING.txt  conf  logs	    temp     webapps.dist
CONTRIBUTING.md  NOTICE   RELEASE-NOTES  bin	      lib   native-jni-lib  webapps  work
```

#### Docker 安装 ElasticSearch + Kibana

```shell
# 官方的使用
docker run -d --name elasticsearch -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch:7.6.2

# es 是十分耗内存的 1.xG 服务器 2核2G

# 查看 docker stats
[root@ouwen666 home]# docker stats
CONTAINER ID   NAME            CPU %     MEM USAGE / LIMIT     MEM %     NET I/O   BLOCK I/O
b496914b7726   elasticsearch   0.00%     1.237GiB / 1.694GiB   73.00%    0B / 0B   197MB / 729kB

# 测试一下es是否安装成功
[root@ouwen666 home]# curl localhost:9200
{
  "name" : "b496914b7726"，
  "cluster_name" : "docker-cluster"，
  "cluster_uuid" : "v5CISdg4Sw-d8-Jui-XXTw"，
  "version" : {
    "number" : "7.6.2"，
    "build_flavor" : "default"，
    "build_type" : "docker"，
    "build_hash" : "ef48eb35cf30adf4db14086e8aabd07ef6fb113f"，
    "build_date" : "2020-03-26T06:34:37.794943Z"，
    "build_snapshot" : false，
    "lucene_version" : "8.4.0"，
    "minimum_wire_compatibility_version" : "6.8.0"，
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  }，
  "tagline" : "You Know， for Search"
}

# 增加对内存的限制 修改配置文件 -e 环境配置的修改
docker run -d --name elasticsearch01 -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -e ES_JAVA_OPTS="-Xms64m -Xmx512m" elasticsearch:7.6.2

# 查看 docker stats 内存明显变小
[root@ouwen666 home]# docker stats
CONTAINER ID   NAME              CPU %     MEM USAGE / LIMIT     MEM %    NET I/O      BLOCK I/O
c0d59f8ca889   elasticsearch01   0.00%     375.2MiB / 1.694GiB   21.63%   524B / 942B  107MB/733kB
# 测试是否启动成功
[root@ouwen666 home]# curl localhost:9200
{
  "name" : "c0d59f8ca889"，
  "cluster_name" : "docker-cluster"，
  "cluster_uuid" : "ECE4OHoqQ5Sk-fhT-ALuPg"，
  "version" : {
    "number" : "7.6.2"，
    "build_flavor" : "default"，
    "build_type" : "docker"，
    "build_hash" : "ef48eb35cf30adf4db14086e8aabd07ef6fb113f"，
    "build_date" : "2020-03-26T06:34:37.794943Z"，
    "build_snapshot" : false，
    "lucene_version" : "8.4.0"，
    "minimum_wire_compatibility_version" : "6.8.0"，
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  }，
  "tagline" : "You Know， for Search"
}
```

如何使用 Kibana 连接 ES？

![image-20211218201949070](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20211218201949.png)

## Docker 可视化

- portainer

- Rancher

### 1. 什么是 portainer？

Docker 图形化界面管理工具！提供一个后台面板供我们操作！

```shell
docker run -d -p 8088:9000 --restart=always -v /var/run/docker.sock:/var/run/docker.sock --privileged=true portainer/portainer
```

访问测试：http://ip:8088/

![image-20211218203604987](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20211218203605.png)

设置登录密码，创建用户

![image-20211218203652466](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20211218203652.png)

管理本地环境

![image-20211218203840461](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20211218203840.png)

## Docker 镜像详解

### 1. 镜像是什么

镜像是一种轻量级、可执行的独立软件包，它包含运行某个软件所需的所有内容，我们把应用程序和配置依赖打包好形成一个可交付的运行环境(包括代码、运行时需要的库、环境变量和配置文件等)，这个打包好的运行环境就是image镜像文件。

只有通过这个镜像文件才能生成Docker容器实例(类似Java中new出来一个对象)。

如何得到镜像：

- 从远程仓库下载
- 自己制作一个镜像 DockerFile

### 2. Docker 镜像加载原理

> UnionFS(联合文件系统)

Union 文件系统(UnionFS) 是一种分层. 轻量级并且高性能的文件系统，他支持对文件系统的修改作为一次提交来层层的叠加，同时可以将不同目录挂载到同一个虚拟文件系统下（unite several directories into a single virtual filesystem）。Union 文件系统是 Docker 镜像的基础。==镜像可以通过分层来进行继承==，基于基础镜像（没有父镜像），可以制作各种具体的应用镜像。

特性：一次同时加载多个文件系统，但从外面看起来，只能看到一个文件系统，联合加载会把各层文件系统叠加起来，这样最终的文件系统会包含所有底层文件和目录。

> Docker 镜像加载原理

docker 的镜像实际上由一层一层的文件系统组成，这种层级的文件系统 UnionFS。

bootfs(boot file system) 主要包含 bootloader 和 kernel，bootloader 主要是引导加载 kernel，Linux 刚启动时会加载 bootfs 文件系统，==在 Docker 镜像的最底层是 bootfs==。这一层与我们典型的 Linux/Unix 系统是一样的，包含 boot 加载器和内核。当 boot 加载完成之后整个内核就存在内存中了，此时内存的使用权已由 bootfs 转交给内核，此时系统也会卸载 bootfs。

roorfs （root file system），在 bootfs 之上。包含的就是典型 Linux 系统中的 /dev ，/proc，/bin ，/etx 等标准的目录和文件。rootfs 就是各种不同的操作系统发行版。比如 Ubuntu，Centos 等等。

![image-20211218204700594](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20211218204700.png)

平时安装进虚拟机的 CentOS 镜像都是好几个 G，为什么 Docker 这里才 200M？

![image-20211218204834885](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20211218204834.png)

对于一个精简的 OS，rootfs 可以很小，只需要包括最基本的命令. 工具和程序库就可以了，因为底层直接用 Host（宿主机）的 kernel，自己只需要提供 rootfs 就行了，由此可见对于不同的 Linux 发行版，bootfs 基本是一致的，rootfs 会有差别，因此不同的发行版可以公用 bootfs。

### 3. 分层的镜像

下载一个镜像，观察下载的日志，可以发现是一层一层往下下载的！

```shell
[root@ouwen666 home]# docker pull redis
Using default tag: latest
latest: Pulling from library/redis
e5ae68f74026: Already exists
37c4354629da: Pull complete
b065b1b1fa0f: Pull complete
6954d19bb2e5: Pull complete
6333f8baaf7c: Pull complete
f9772c8a44e7: Pull complete
Digest: sha256:2f502d27c3e9b54295f1c591b3970340d02f8a5824402c8179dcd20d4076b796
Status: Downloaded newer image for redis:latest
docker.io/library/redis:latest
```

> 为什么 Docker 镜像要采用这种分层的结构？

==最大的好处，就是是资源共享了！==

如有多个镜像都从相同的基本镜像构建而来，那么宿主机只需在磁盘上保留一份基本镜像，同时内存中也只需要加载一份基本镜像 ，这样就可以为所有的容器服务了，且镜像的每一层都可以被共享。

查看镜像分层的方式可以通过`docker image inspect` 命令！

```shell
[root@ouwen666 home]# docker image inspect redis
[
...
        "RootFS": {
            "Type": "layers"，
            "Layers": [
                "sha256:9321ff862abbe8e1532076e5fdc932371eff562334ac86984a836d77dfb717f5"，
                "sha256:aa2858ea5edc9c0981901a1b63b49a8f4a6e7099b4304b49e680ffdcc6b71b3e"，
                "sha256:93079bf13a6d5fe7c4bd9f00cb96183f9d1db9968c4bd15b395df2f3867bf8e5"，
                "sha256:9ca504b88e256aa6f6c04ec65aeeed6b926661ea30a0b97f829fbe230155241a"，
                "sha256:9468a3f0498bd5cc298ce25ea6ce9c6adf14aa2ce152856b5f389510a9bb9e01"，
                "sha256:b7851a62867d82784052d7662862adc0b47b2bddcddc89ae78307f75ba1b29ae"
            ]
        }
...
]
```

> 理解

所有的 Docker 镜像都起始于一个基础镜像层 ，当进行修改或增加新的内容时，就会在当前镜像层之上，创建新的镜像层。

举一个简单的例子，假如基于 Ubuntu Linux 16.04 创建一个新的镜像 ，这就是新镜像的第一层。如果在该镜像中添加 Python 包，就会在基础镜像层之上创建第二个镜像层;如果继续添加一个安全补丁，就会创建第三个镜像层。

该镜像当前已经包含 3 个镜像层，如下图所示(这只是一个用于演示的很简单的例子 )。

![image-20211218210258974](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20211218210259.png)

在添加额外的镜像层的同时，镜像始终保持是当前所有镜像的组合，理解这一点非常重要。

下图中举 了一个简单的例子，每个镜像层包含 3 个文件，而镜像包含了来自两个镜像层的 6 个文件。

![image-20211218210445762](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20211218210445.png)

上图中的镜像层跟之前图中的略有区别，主要目的是便于展示文件。

下图中展示了一个稍微复杂的三层镜像，在外部看来整个镜像只有 6 个文件，这是因为最上层中的文件 7 是文件 5 的一一个更新版本。

![image-20211218210534319](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20211218210534.png)

这种情况下，上层镜像层中的文件覆盖了底层镜像层中的文件。这样就使得文件的更新版本作为一个新镜像层添加到镜像当中。

Docker 通过存储引擎(新版本采用快照机制)的方式来实现镜像层堆栈，并保证多镜像层对外展示为统一的文件系统。

Linux 上可用的存储引擎有 AUFS. Overlay2. Device Mapper. Btrfs 以及 ZFS。顾名思义，每种存储引擎都基于 Linux 中对应的文件系统或者块设备技术，并且每种存储引擎都有其独有的性能特点。

Docker 在 Windows 上仅支持 windowsfilter -种存储引擎，该引擎基于 NTFS 文件系统之上实现了分层和 CoW[1]。

下图展示了与系统显示相同的三层镜像。所有镜像层堆叠并合并，对外提供统一的视图。

![image-20211218210605289](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20211218210605.png)

> 特点

Docker 镜像都是只读的，当容器启动时，一个新的可写层被加载到镜像的顶部！这一层就是我们通常说的容器层，容器之下都叫镜像层！

### 4. 提交镜像

```shell
docker commit 提交容器成为一个新的镜像副本

# 命令和git原理类似
docker commit -m="提交的描述信息" -a="作者" 容器ID 目标镜像名:[标签名]
```

**实战测试**

```shell
# 启动一个默认的tomcat

# 发现默认的tomcat的webapps目录下是没用部署应用的。原因是官方的镜像都是默认webapps下是没有应用的

# 将webapps.dist目录下的应用拷贝到webapps下

# 将更改过的镜像提交到仓库，以后就能使用修改过的镜像进行启动！
[root@ouwen666 ~]# docker commit -a="Irving" -m="add webapps app" 9bffc8b128c7 mytomcat:1.0
sha256:ef1ba8ee4bba1a39202b89a9bfecc4cb4dfbf20263b6e1b913a4cecf80ff8381
[root@ouwen666 ~]# docker images
REPOSITORY            TAG       IMAGE ID       CREATED          SIZE
mytomcat              1.0       ef1ba8ee4bba   10 seconds ago   685MB
tomcat                9.0       3f3cadde9a68   10 days ago      680MB
redis                 latest    aea9b698d7d1   2 weeks ago      113MB
nginx                 latest    f652ca386ed1   2 weeks ago      141MB
centos                latest    5d0da3dc9764   3 months ago     231MB
portainer/portainer   latest    580c0e4e98b0   9 months ago     79.1MB
elasticsearch         7.6.2     f29a1ee41030   21 months ago    791MB
```

### 5. 提交镜像到阿里云

本地镜像发布到阿里云流程

![image-20220604164136425](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220612132649.png)



1. 登录阿里云控制台，选择容器镜像服务

![image-20220604211107790](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220604211156.png)

2. 选择个人实例

![image-20220604211320410](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220604211320.png)

3. 创建命名空间

![image-20220604211650899](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220604211651.png)

4. 创建镜像仓库

![image-20220604212038508](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220604212038.png)

5. 继续

![image-20220604212532015](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220604212554.png)

6. 继续

![image-20220604212648063](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220604212648.png)

7. 进入管理界面获取脚本

![image-20220604212843596](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220604212843.png)

8. 将镜像推送到阿里云

![image-20220604213052191](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220604213052.png)

### 6. 提交镜像到私有库

Docker Registry是官方提供的工具，可以用于构建私有镜像仓库。

1. **下载镜像Docker Registry**

```bash
[root@88231 ~]# docker pull registry
Using default tag: latest
latest: Pulling from library/registry
79e9f2f55bf5: Pull complete 
0d96da54f60b: Pull complete 
5b27040df4a2: Pull complete 
e2ead8259a04: Pull complete 
3790aef225b9: Pull complete 
Digest: sha256:169211e20e2f2d5d115674681eb79d21a217b296b43374b8e39f97fcf866b375
Status: Downloaded newer image for registry:latest
```

2. **运行私有库 Registry，相当于自己搭建一个 Docker Hub**

默认情况，仓库被创建在容器内的 /var/lib/registry 目录下，建议自行用容器卷映射，方便于宿主机联调

```bash
[root@88231 hd]# docker run -d -p 5000:5000  -v /hd/docker-registry/:/var/lib/registry --privileged=true registry
96579e94a32238269d25239394f62a7d38492d27834ebb4863fe8d3baea55b77
```

3. **curl验证私服库上有什么镜像**

`curl -XGET http://192.168.88.231:5000/v2/_catalog`

可以看到，目前私服库没有任何镜像上传过

```bash
[root@88231 docker-registry]# curl -XGET http://192.168.88.231:5000/v2/_catalog
{"repositories":[]}
```

4. **提交一个新镜像到私有的 Registry 库**

- 修改符合私服规范的 Tag

`docker tag 镜像ID Host:Port/Repository:Tag`

```
[root@88231 docker-registry]# docker tag ba6acccedd29 192.168.88.231:5000/myubuntu:1.0
```

- 修改配置文件，使之支持 http 上传

这里的地址是 registry 私服所在主机的地址

```bash
[root@88231 docker-registry]# vim /etc/docker/daemon.json 
{
  "registry-mirrors": ["https://alq7pwwu.mirror.aliyuncs.com"],
  "insecure-registries": ["192.168.88.231:5000"]
}
# 重启 docker 生效
[root@88231 docker-registry]# service docker restart
Stopping docker:                                       [  OK  ]
Starting docker:	                                   [  OK  ]
```

- push 推送到私服库

```
# 使用 docker push 命令推送私服库
[root@88231 docker-registry]# docker push 192.168.88.231:5000/myubuntu:1.0
The push refers to a repository [192.168.88.231:5000/myubuntu]
9f54eef41275: Pushed 
1.0: digest: sha256:870c68e5f7e5cac7cb9a747e18865524dbc0952575dcc498621c79b94a78a846 size: 529

# 使用 curl 查看私服库上的镜像
[root@88231 docker-registry]# curl -XGET http://192.168.88.231:5000/v2/_catalog
{"repositories":["myubuntu"]}
```

5. **将私服库上的镜像 pull 到本地运行**

```
# 使用 docker pull 命令将私服库上的镜像下载到本地
[root@88231 /]# docker pull 192.168.88.231:5000/myubuntu:1.0
1.0: Pulling from myubuntu
f9945daba3cc: Pull complete 
Digest: sha256:870c68e5f7e5cac7cb9a747e18865524dbc0952575dcc498621c79b94a78a846
Status: Downloaded newer image for 192.168.88.231:5000/myubuntu:1.0
```

## 容器数据卷

### 1. 什么是容器数据卷？

> docker 的理念回顾

将应用和环境打包成一个镜像！

数据都在容器中，如果删除容器，数据就会丢失！==数据如何持久化？数据需要存储在本地！==

容器之间可以有一个数据共享的技术！Docker 容器中产生的数据，可以同步到本地！

这就是卷技术！其本质就是目录的挂载，将容器内的目录，挂载到 Linux 上！

### 2. 使用数据卷

> ==坑！使用容器数据卷时记得加入：--privileged=true==
>
> Docker挂载主机目录访问如果出现cannot open directory: Permission denied
> 解决办法：在挂载目录后多加一个--privileged=true参数即可
>
> 如果是CentOS7安全模块会比之前系统版本加强，不安全的会先禁止，所以目录挂载的情况被默认为不安全的行为，
> 在SELinux里面挂载目录被禁止掉了额，如果要开启，我们一般使用--privileged=true命令，扩大容器的权限解决挂载目录没有权限的问题，也即使用该参数，container内的root拥有真正的root权限，否则，container内的root只是外部的一个普通用户权限。

> 方式一：直接使用命令来挂载 -v

```shell
docker run -it -v 主机目录:容器内目录

# 测试
[root@ouwen666 ~]# docker run -it -v /home/ceshi:/home --privileged=true centos /bin/bash

# 启动成功后查看详细信息
[root@ouwen666 ceshi]# docker inspect 015ee9a39cf1
```

![image-20211219162714621](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20211219162714.png)

测试文件的同步

![image-20211219163426342](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20211219163426.png)

继续测试！

1. 停止容器
2. 宿主机上修改文件
3. 启动并进入容器
4. 发现容器内的数据跟宿主机是同步的！

![image-20211219163920495](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20211219163920.png)

> 好处：修改只需要在本地修改即可，容器内会自动同步！

### 3. 实战：安装 MySQL

思考：MySQL 的数据持久化问题！

```shell
# 获取镜像
[root@ouwen666 ~]# docker pull mysql:5.7

# 运行容器，需要做数据挂载！ # 安装启动mysql，需要配置密码的
# 官方命令: docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag

# 启动mysql
docker run -d -p 3306:3306 -v /home/mysql/conf:/etc/mysql/conf.d -v /home/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 --name mysql01 mysql:5.7
# 命令解释
-d  后台运行
-p  端口映射
-v  卷挂载
-e  环境配置
--name 容器名字

# 启动成功，通过Navicat连接测试！
```

![image-20211219171950528](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20211219171950.png)

用 Navicat 建一个数据库

![image-20211219193046702](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20211219193046.png)

服务器上映射的路径下，出现了同名文件

![image-20211219193150622](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20211219193150.png)

将容器删除，发现 `/home/mysql/data` 目录下的文件还是存在的。这就实现了容器数据持久化！

![image-20211219193417029](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20211219193417.png)

### 4. 具名和匿名挂载

```shell
# 匿名挂载
-v 容器内路径
docker run -d -P --name nginx01 -v /etc/nginx nginx

# 查看所有 volume 的情况
[root@ouwen666 data]# docker volume ls
DRIVER    VOLUME NAME
local     5370f027b4d5a86a9718f66c9bc9c39138aa92ad2b6368a74f930c09f94c52bb

# 这里发现 volume name 是一串乱码，是因为我们挂载时没有指定名字。这就是匿名挂载。我们在 -v 时只写了容器内路径，没有写容器外路径！

# 具名挂载
[root@ouwen666 data]# docker run -d -P --name nginx02 -v nginxvolumename:/etc/nginx nginx
11ba9ffded8187484386ff37103c91a6a2bd2e103420b9376c45c61f604dab57
[root@ouwen666 data]# docker volume ls
DRIVER    VOLUME NAME
local     nginxvolumename

# 通过 -v 卷名:容器内路径 完成具名挂载
# 查看这个卷的具体信息 inspect
[root@ouwen666 data]# docker volume inspect nginxvolumename
[
    {
        "CreatedAt": "2021-12-19T19:44:11+08:00",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/nginxvolumename/_data",
        "Name": "nginxvolumename",
        "Options": null,
        "Scope": "local"
    }
]
```

> 所有的 docker 容器内的卷，没有指定目录的情况下都是在 `/var/lib/docker/volumes/xxx/_data` 下！
>
> 通过具名挂载可以方便地找到卷所在的位置，大多数请况下使用 `具名挂载` ！

![image-20211219194814407](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20211219194814.png)

**如何确定是具名挂载. 匿名挂载还是指定路径挂载？**

- `-v 容器内路径` 匿名挂载
- `-v 卷名:容器内路径` 具名挂载
- `-v /宿主机路径:容器内路径` 指定路径挂载

拓展：

```shell
# 通过 -v 容器内路径:ro/rw  改变读写权限
ro   readonly  # 只读
rw   readwrite # 可读可写

# 例如
docker run -d -P --name nginx02 -v nginxvolumename:/etc/nginx:ro nginx
docker run -d -P --name nginx02 -v nginxvolumename:/etc/nginx:rw nginx

# 一旦设置了ro，就说这个路径只能通过映射后宿主机的来操作，容器内部是无法操作的！
```

### 5. 初识 DockerFile

DockerFile 就是用来构建 docker 镜像的构建文件！其实就是一段命令脚本！

通过这个脚本可以生成镜像，镜像是一层一层的，脚本是一个个的命令，每个命令都是一层！

```shell
# 创建一个dockerfile文件，名字可以随意 建议 Dockerfile
[root@ouwen666 docker-test]# vim Dockerfile
FROM centos

VOLUME ["volume01","volume02"]

CMD echo "-----end-----"

CMD /bin/bash

# 通过Dockerfile构建一个属于自己的镜像 注意末尾有一个 . 代表当前路径
[root@ouwen666 docker-test]# docker build -f /home/docker-test/Dockerfile -t irving/centos:1.0 .
Sending build context to Docker daemon  2.048kB
Step 1/4 : FROM centos
 ---> 5d0da3dc9764
Step 2/4 : VOLUME ["volume01","volume02"]
 ---> Running in e121337bcbbe
Removing intermediate container e121337bcbbe
 ---> 2e6cad23ca38
Step 3/4 : CMD echo "-----end-----"
 ---> Running in 7025e750f7ac
Removing intermediate container 7025e750f7ac
 ---> cf376f17795b
Step 4/4 : CMD /bin/bash
 ---> Running in 60e0bccacc5d
Removing intermediate container 60e0bccacc5d
 ---> 2aee0e7445ac
Successfully built 2aee0e7445ac
Successfully tagged irving/centos:1.0

# 查看生成的镜像
[root@ouwen666 docker-test]# docker images
REPOSITORY            TAG       IMAGE ID       CREATED         SIZE
irving/centos         1.0       2aee0e7445ac   2 minutes ago   231MB
mytomcat              1.0       ef1ba8ee4bba   4 hours ago     685MB
tomcat                9.0       3f3cadde9a68   10 days ago     680MB
redis                 latest    aea9b698d7d1   2 weeks ago     113MB
mysql                 5.7       738e7101490b   2 weeks ago     448MB
nginx                 latest    f652ca386ed1   2 weeks ago     141MB
centos                latest    5d0da3dc9764   3 months ago    231MB
portainer/portainer   latest    580c0e4e98b0   9 months ago    79.1MB
elasticsearch         7.6.2     f29a1ee41030   21 months ago   791MB

# 使用刚刚生成的镜像启动一个容器
[root@ouwen666 docker-test]# docker run -it 2aee0e7445ac /bin/bash
[root@b2707d29bda4 /]#
```

![image-20211219203120341](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20211219203120.png)

这个挂载的卷目录一定和外部有一个同步的目录！

通过 `docker inspect 容器ID` 查看具体信息

![image-20211219203438140](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20211219203438.png)

发现确实是在 `/var/lib/docker/volumes/` 目录下的一个随机目录下！

### 6. 卷的继承和共享

> 图解

![image-20211219203754331](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20211219203754.png)

> 测试！

```shell
# 启动三个容器！通过刚刚自己制作的镜像启动

# 第一个容器 docker01
[root@ouwen666 ~]# docker run -it --name docker01 irving/centos:1.0
[root@9e033da9de3e /]# ls
bin  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var  volume01	volume02

# 第二个容器 docker02 通过 --volumes-from 挂载 docker01 容器
[root@ouwen666 ~]# docker run -it --name docker02 --volumes-from docker01 irving/centos:1.0
[root@75fb856af436 /]# ls
bin  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var  volume01	volume02

# docker01 创建的文件同步到 docker02 容器上了
[root@ouwen666 ~]# docker attach docker01
[root@9e033da9de3e /]# ls
bin  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var  volume01	volume02
[root@9e033da9de3e /]# cd volume01
[root@9e033da9de3e volume01]# ls
[root@9e033da9de3e volume01]# touch docker01
[root@9e033da9de3e volume01]# ls
docker01

[root@ouwen666 /]# docker attach docker02
[root@75fb856af436 /]# ls
bin  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var  volume01	volume02
[root@75fb856af436 /]# cd volume01
[root@75fb856af436 volume01]# ls
docker01

# 第三个容器 docker03 也通过 --volumes-from 挂载 docker01
[root@ouwen666 ~]# docker run -it --name docker03 --volumes-from docker01 irving/centos:1.0
[root@eda4d0cad3f0 /]# ls
bin  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var  volume01	volume02
[root@eda4d0cad3f0 /]# cd volume01
[root@eda4d0cad3f0 volume01]# ls
docker01

# 发现文件还是同步过来，在 docker03 中新建一个文件
[root@eda4d0cad3f0 volume01]# touch docker03
[root@eda4d0cad3f0 volume01]# ls
docker01  docker03

# 进入 docker01 容器，发现 docker03 中创建的文件也同步过来了！
[root@ouwen666 /]# docker attach docker01
[root@9e033da9de3e /]# cd volume01
[root@9e033da9de3e volume01]# ls
docker01  docker03
```

> 结论

只要通过 `--volumes-from` 就可以做到容器间的数据共享！

思考：删除 docker01，查看 docker02. docker03 是否还能访问这些文件

依旧可以访问！本质上是一种数据拷贝，而不是单纯的数据共享！

容器之间配置信息的传递，数据卷容器的生命周期可以一直持续到没有人使用为止！

