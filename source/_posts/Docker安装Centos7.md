---
title: Docker安装Centos7
tags:
  - Docker
  - 容器
categories: Linux
cover: 'https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20211212170545.png'
abbrlink: e33c148b
date: 2021-12-12 16:25:44
updated: 2022-01-05 00:50:38
---

# Docker 安装 Centos7

## 前言

本地化启动一个 `Linux` 服务器，使用 `Docker` 容器来构建一个包含SSH服务的 Linux 镜像。

## 准备centos官方镜像

具体步骤：

```shell
# 1、拉取centos7官方镜像
docker pull centos:7
# 2、启动镜像
docker run -itd --privileged centos:7 init
# 3、进入容器bash中
docker exec -it 镜像ID bash
# 4、修改root用户密码
[root@33c7ee24f43e /]# passwd
Changing password for user root.
New password:
BAD PASSWORD: The password is shorter than 8 characters
Retype new password:
passwd: all authentication tokens updated successfully.
# 5、安装和配置ssh服务
yum install openssh-server -y
# 6、修改/etc/ssh/sshd_config配置并保存：PermitRootLogin yes    UsePAM no
```

![image-20220223145827608](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220223145834.png)	

![image-20220223145949729](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220223145949.png)	

```shell
# 7、启动ssh服务
[root@33c7ee24f43e /]# systemctl start sshd
# 8、退出容器
[root@33c7ee24f43e /]# exit
exit
```

## 构建镜像

通过 `docker commit` 来构建包含 ssh 服务的镜像

```shell
# 1、通过commit构建镜像
docker commit -a="Irving" -m="本地linux镜像" 容器ID mycentos:1.0
# 2、启动镜像 -p 10000:22 映射本地10000端口
docker run -d -p 10000:22 --name 容器名 镜像ID /usr/sbin/sshd -D
```

## 使用xshell进行连接

```shell
ssh://root:****@127.0.0.1:10000
```



