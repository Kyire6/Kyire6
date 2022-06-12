---
title: GitLab安装与使用
tags:
  - Docker
  - 容器
  - Git
categories: Linux
cover: 'https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220111153032.png'
abbrlink: 3653ea16
date: 2022-01-11 15:26:59
updated: 2022-01-11 15:26:59
---

# GitLab 安装与使用

## Docker 安装 GitLab

DockerHub 上有许多制作完善的镜像，直接搜索 `gitlab` 查看镜像：

**搜索镜像**

```shell
docker search gitlab
```

![image-20220111153600801](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220111153600.png)

> 这里可以选择汉化的 GitLab 社区版的镜像进行下载

**下载镜像**

```shell
docker pull twang2218/gitlab-ce-zh
```

**启动镜像**

```shell
docker run -d -p 8443:443 -p 8090:80 -p 8022:22 --restart always --name gitlab -v /usr/local/gitlab/etc:/etc/gitlab -v /usr/local/gitlab/log:/var/log/gitlab -v /usr/local/gitlab/data:/var/opt/gitlab --privileged=true twang2218/gitlab-ce-zh

命令解释：
-d 后台启动
-p 映射端口
--restart 重启配置
-v 卷挂载
--privileged=true 设置root权限
```

## 配置 GitLab

**编辑 `gitlab.rb` 文件**

```shell
# 直接修改 -v 挂载后的目录，docker会自动同步到容器内部
vim /usr/local/gitlab/etc/gitlab.rb

# 修改ip和端口 可以使用 / 来定位配置项
# http地址 -- 无需配置端口
external_url 'http://xx.xx.xx.xx'
# ssh地址 -- 无需配置端口
gitlab_rails['gitlab_ssh_host'] = 'xx.xx.xx.xx'(不用添加端口)
# ssh端口 默认22 启动容器时我们映射为8022
gitlab_rails['gitlab_shell_ssh_port'] = 8022

# ========== 邮箱配置 ==============
# 是否启用
gitlab_rails['smtp_enable'] = true
# SMTP服务的地址
gitlab_rails['smtp_address'] = "smtp.qq.com"
# 端口
gitlab_rails['smtp_port'] = 465
# 你的QQ邮箱（发送账号）
gitlab_rails['smtp_user_name'] = "958317640@qq.com"
# 授权码
gitlab_rails['smtp_password'] = "********"
# 域名
gitlab_rails['smtp_domain'] = "smtp.qq.com"
# 登录验证
gitlab_rails['smtp_authentication'] = "login"

# 使用了465端口，就需要配置下面三项
gitlab_rails['smtp_enable_starttls_auto'] = true
gitlab_rails['smtp_tls'] = true
gitlab_rails['smtp_openssl_verify_mode'] = 'none'

# 你的QQ邮箱（发送账号）
gitlab_rails['gitlab_email_from'] = '958317640@qq.com'
```

**应用配置**

```shell
# 注意观察日志输出
gitlab-ctl reconfigure
```

![image-20220111154833139](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220111154833.png)

**编辑 `gitlab.yml`**

```shell
# 进入容器
docker exec -it gitlab bash
# 编辑yml配置
vim /opt/gitlab/embedded/service/gitlab-rails/config/gitlab.yml
# 修改port为启动时映射的端口
port: 8090

# 重启服务并测试
gitlab-ctl restart
```

![image-20220111155220504](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220111155220.png)

