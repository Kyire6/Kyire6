---
title: Nginx从入门到实战
tags:
  - 笔记
  - 技巧
  - Nginx
categories: 服务器
cover: 'https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20211212174004.png'
abbrlink: 3db3af36
date: 2022-04-24 01:27:18
updated: 2022-04-24 01:27:18
---

# Nginx从入门到实战

## Nginx的安装

本文Linux环境基于`centos7`

### 版本区别

常用版本分为四大阵营

- Nginx开源版 http://nginx.org/ 
- Nginx plus 商业版 https://www.nginx.com 
- openresty http://openresty.org/cn/ 
- Tengine http://tengine.taobao.org/

### 编译安装

这里下载的是 `nginx-1.21.6.tar.gz` ，解压后编译安装

```bash
# 解压
tar zxvf nginx-1.21.6.tar.gz -C ./
cd nginx-1.21.6
# 执行配置脚本
./configure --prefix=/usr/local/nginx
# 安装
make && make install
```

### 如果出现警告或报错

一般是缺少依赖的问题

```bash
# 安装gcc
yum install -y gcc
# 安装perl库
yum install -y pcre pcre-devel
# 安装zlib库
yum install -y zlib zlib-devel
# 重新执行安装
make && make install
```

### 安装成系统服务

1. 创建服务脚本

`vi /usr/lib/systemd/system/nginx.service`

2. 服务脚本内容

```
[Unit]
Description=nginx - web server
After=network.target remote-fs.target nss-lookup.target
[Service]
Type=forking
PIDFile=/usr/local/nginx/logs/nginx.pid
ExecStartPre=/usr/local/nginx/sbin/nginx -t -c /usr/local/nginx/conf/nginx.conf
ExecStart=/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
ExecReload=/usr/local/nginx/sbin/nginx -s reload
ExecStop=/usr/local/nginx/sbin/nginx -s stop
ExecQuit=/usr/local/nginx/sbin/nginx -s quit
PrivateTmp=true
[Install]
WantedBy=multi-user.target
```

3. 重新加载系统服务

`systemctl daemon-reload`

4. 启动服务

`systemctl start nginx.service`

5. 开机自启

`systemctl enable nginx.service`

## Nginx基础使用

### 目录结构

进入`Nginx`的主目录可以看到这些文件夹

```
client_body_temp conf fastcgi_temp html logs proxy_temp sbin scgi_temp uwsgi_temp
```

其中这几个文件夹在刚安装时是没有的， 主要用来存放运行过程中的临时文件

```
client_body_temp fastcgi_temp proxy_temp scgi_temp
```

- conf：用来存放配置文件
- html：用来存放静态文件的默认目录 html、css等
- sbin：nginx的主程序
- logs：nginx运行日志

### 基本运行原理

![image-20220424022053512](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220424022100.png)	

## Nginx配置

### 最小配置

```
#  默认为1，表示开启一个业务进程
worker_processes  1;

events {
	# 单个业务进程可接受连接数
    worker_connections  1024;
}


http {
	# 引入http mime类型
    include       mime.types;
    # 如果mime类型没匹配上，默认使用二进制流的方式传输。
    default_type  application/octet-stream;
	使用linux的 sendfile(socket, file, len) 高效网络传输，也就是数据0拷贝。
    sendfile        on;

    keepalive_timeout  65;

	# 虚拟主机 vhost
    server {
    	# 监听端口号
        listen       80;
        # 域名、主机名
        server_name  localhost;

		# 匹配路径
        location / {
            root   html; # 文件根目录
            index  index.html index.htm; # 默认页名称
        }

		# 报错编码对应页面
        error_page   500 502 503 504  /50x.html; 
        location = /50x.html {
            root   html;
        }
    }

}
```

### 虚拟主机

原本一台服务器只能对应一个站点，通过虚拟主机技术可以虚拟化成多个站点同时对外提供服务

**server_name匹配规则**

我们需要注意的是 `server_name` 匹配分先后顺序，写在前面的匹配上就不会继续往下匹配了。

- 完整匹配

可以在同一个 `server_name` 中配置多个域名

```
server_name abc.com abc123.com;
```

- 通配符匹配

可以通过 `*` 通配符来模糊匹配多个域名，可以在开始和结尾使用

```
server_name *.abc.com abc.*
```

- 正则匹配

```
server_name ~^[0-9]+\.abc\.com$
```

### 反向代理

通过关键字 `proxy_pass` 关键字来指定一个服务器地址（ip/域名）

```nginx
location / {
	proxy_pass http://www.baidu.com/;
}
```

#### 负载均衡

-  基于反向代理的负载均衡配置

```nginx
upstream app {
    server 192.168.88.102:80;
    server 192.168.88.103:80;
}

location / {
	proxy_pass http://app;
}
```

#### 负载均衡策略

- **轮询**

默认情况下使用轮询方式，逐一转发，这种方式适用于无状态请求

- **权重（weight）**
  - down：表示当前的主机暂时不参与负载 
  - weight：默认为1，weight越大，负载的权重就越大
  - backup： 其它所有的非backup机器down或者忙的时候，请求backup机器

指定轮询几率，weight和访问比率成正比，用于后端服务器性能不均的情况。

```nginx
upstream app {
    server 192.168.88.102:80 weight=10 down;
    server 192.168.88.103:80 weight=1;
    server 127.0.0.1:8060 weight=1 backup;
}
```

- **ip_hash**

根据客户端的ip地址转发同一台服务器，可以保持回话

- **least_conn**

最少连接数访问

- **url_hash**

根据用户访问的url定向转发请求

- **fair**

根据后端服务器响应时间转发请求

### 动静分离

- 配置后端服务的反向代理

```nginx
location / {
	proxy_pass http://127.0.0.1:8080;
}
```

- 配置前端静态资源

```nginx
location /css {
    root /usr/local/nginx/static;
    index index.html index.htm;
}

location /images {
    root /usr/local/nginx/static;
    index index.html index.htm;
}

location /js {
    root /usr/local/nginx/static;
    index index.html index.htm;
}

# 正则匹配
location ~*/(css|img|js) {
    root /usr/local/nginx/static;
    index index.html index.htm;
}
```

### location配置规则

#### location前缀

- / 通用匹配，任何请求都会匹配到

- = 精准匹配，不是以指定模式开头 

- ~ 正则匹配，区分大小写 

- ~* 正则匹配，不区分大小写
- ^~ 非正则匹配，匹配以指定模式开头的location

#### location匹配规则

- 多个正则 `location` 直接按书写顺序匹配，匹配成功后就不会往下匹配
- 普通（非正则）`location` 会一直往下，直到找到匹配度最高的（最大前缀匹配）
- 当普通 `location` 与正则 `location` 同时存在，如果正则匹配成功,则不会再执行普通匹配
- 所有类型 `location` 存在时，`=匹配` > `^~匹配` > `正则匹配` > `普通（最大前缀匹配）`

#### alias与root

`root` 用来设置根目录，而 `alias` 在接受请求的时候在路径上不会加上 `location`

```nginx
location /css {
    alias /usr/local/nginx/static/css;
    index index.html index.htm;
}
```

- alias指定的目录是准确的，即location匹配访问的path目录下的文件直接是在alias目录下查找的；
- root指定 的目录是location匹配访问的path目录的上一级目录,这个path目录一定要是真实存在root指定目录下的；
- 使用 alias标签的目录块中不能使用rewrite的break（具体原因不明）；另外，alias指定的目录后面必须要加上"/"符 号！！ 
- alias虚拟目录配置中，location匹配的path目录如果后面不带"/"，那么访问的url地址中这个path目录后 面加不加"/"不影响访问，访问时它会自动加上"/"； 但是如果location匹配的path目录后面加上"/"，那么访问的url地 址中这个path目录必须要加上"/"，访问时它不会自动加上"/"。如果不加上"/"，访问就会失败！ 
- root目录配置中，location匹配的path目录后面带不带"/"，都不会影响访问。

### URLRewrite

rewirte语法格式及参数语法：

`rewrite` 是实现URL重写的关键指令，根据 `regex (正则表达式)` 部分内容，重定向到 `replacement`，结尾是 `flag` 标记。

```
rewrite   <regex>   <replacement>   [flag];
关键字     正则       替代内容         flag标记
```

- **关键字**：其中关键字 `rewrite` 不能改变
- **正则**：正则表达式语句进行规则匹配
- **替代内容**：将正则匹配的内容替换成 `replacement`
- **flag标记**：`rewrite` 支持的 `flag` 标记

> rewrite参数的标签段位置：server、location、if

**flag标记说明：** 

- last：本条规则匹配完成后，继续向下匹配新的location URI规则 
- break：本条规则匹配完成即终止，不再匹配后面的任何规则 
- redirect：返回302临时重定向，浏览器地址会显示跳转后的URL地址 
- permanent：返回301永久重定向，浏览器地址栏会显示跳转后的URL地址

### 防盗链配置

```nginx
valid_referers none | blocked | server_names | strings ....;
```

- none：检测 `Referer` 头域不存在的情况
- blocked：检测 `Referer` 头域的值被防火墙或者代理服务器删除或伪装的情况。这种情况该头域的值不以 `http://` 或 `https://` 开头
- server_names：设置一个或多个 `URL` ，检测 `Referer` 头域的值是否是这些 `URL` 中的某一个。

**使用curl测试**

```
curl -I http://192.168.44.101/img/logo.png

# -I 表示只显示响应的头信息
```

**带引用**

```
curl -e "http://baidu.com" -I http://192.168.44.101/img/logo.png
```

### 高可用配置

#### 安装Keepalived

- 编译安装

下载地址：https://www.keepalived.org/download.html#

```
# 解压之后通过命令安装
./configure

# 如遇报错提示
configure: error:
!!! OpenSSL is not properly installed on your system. !!!
!!! Can not include OpenSSL headers files. !!!

# 安装依赖
yum install openssl-devel
```

- yum安装

```
yum install keepalived
```

#### 配置

使用 `yum` 安装后的配置文件在 `/etc/keepalived/keepalived.conf`

**最小配置**

主机：

```
! Configuration File for keepalived
global_defs {
	router_id lb101
}
vrrp_instance vansys {
    state MASTER
    interface ens33
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
    	192.168.44.200
    }
}
```

从机：

```
! Configuration File for keepalived
global_defs {
	router_id lb100
}
vrrp_instance vansys {
    state MASTER
    interface ens33
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
    	192.168.44.200
    }
}
```

**启动服务**

```
systemctl start keepalived
```

