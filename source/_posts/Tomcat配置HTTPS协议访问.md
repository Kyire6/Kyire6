---
title: Tomcat 配置 HTTPS 协议访问
tags:
  - 笔记
  - Tomcat
categories: 服务器
cover: 'https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20211212174358.png'
abbrlink: b5db2b95
date: 2021-07-15 16:25:44
updated: 2021-07-15 16:26:26
---
# Tomcat 配置 HTTPS 协议访问

## 一、前言

Tomcat 默认支持 HTTP 协议访问，项目需求要修改 Tomcat 支持 HTTPS 协议访问。

## 二、步骤

### 1、使用 Java 自带的 keytool 生成证书

打开控制台输入：

```bash
keytool -genkey -v -alias testKey -keyalg RSA -validity 3650 -keystore D:\apache-tomcat-7.0.26\keys\test.keystore -ext SAN=ip:127.0.0.1

keytool -export -alias testKey -file wxsccp.cer -keystore test.jks
```

参数说明：

```
alias: 别名 这里起名keys
keyalg: 证书算法，RSA
validity：证书有效时间，10年
keystore：证书生成的目标路径和文件名,替换成你自己的路径即可,我定义的是D:\apache-tomcat-7.0.26\keys\test.keystore，其中keys文件夹必须存在
```

之后回车，然后需要输入一些信息，其中秘钥库口令和秘钥口令最好一致，并且记下来（之后配置Tomcat需要用到）

![image-20210607134909105](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210607134909.png)

### 2、配置 Tomcat

打开 Tomcat 的 conf 目录下的 `server.xml `文件，定位 `https` 到指定位置，完整配置如下：

```xml
<Connector port="8443" protocol="org.apache.coyote.http11.Http11Protocol"
               maxThreads="150" SSLEnabled="true" scheme="https" secure="true"
               clientAuth="false" sslProtocol="TLS" 
               keystoreFile="D:\software\EOS\apache-tomcat-7.0.54\keys\test.keystore"
               keystorePass="vansys" />
```

参数说明：

- keystoreFile：上面生成的证书所在目录
- keystorePass：生成证书时输入的秘钥

![image-20210607135533347](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210607135533.png)

添加 http 跳转 https 协议访问，在 web.xml 文件末尾添加如下配置：

```xml
<security-constraint>
    <web-resource-collection >
        <web-resource-name >SSL</web-resource-name>
        <url-pattern>/*</url-pattern>
    </web-resource-collection>
    <user-data-constraint>
        <transport-guarantee>CONFIDENTIAL</transport-guarantee>
    </user-data-constraint>
</security-constraint>
```

如图：

![image-20210607135715701](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210607135715.png)

### 3、重启 Tomcat 服务器

这样就能通过 https 协议访问我们的项目：

![image-20210607140001797](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210607140001.png)

但是发现：

使用自己生成的证书会遇到一些问题：

- **浏览器会对 HTTPS 使用危险标识。**
- **浏览器默认不会加载非HTTPS域名下的javascript**
- **移动设备显示页面空白**

## 三、解决证书无效问题

要解决自己生成的证书无效的问题，需要购买相应的 SSL 证书。[证书服务_SSL数字证书_HTTPS加密_服务器证书_CA认证-阿里云](https://www.aliyun.com/product/cas)

不过我之前用过 一些免费的证书申请网站 [FreeSSL首页 - FreeSSL.cn一个提供免费HTTPS证书申请的网站](https://freessl.cn/)

### 1、申请证书

略

### 2、Tomcat 配置 PFX 证书

打开 Tomcat 配置文件 `conf\server.xml`

定位 `https` 到指定位置，修改三个属性 ：`keystoreFile`，`keystoreType`，`keystorePass`

```xml
  <Connector
        protocol="org.apache.coyote.http11.Http11NioProtocol"
        port="443" maxThreads="200"
        scheme="https" secure="true" SSLEnabled="true"
        clientAuth="false" sslProtocol="TLS"
        keystoreFile="/你的磁盘目录/证书文件名.pfx"  <!--这里是证书文件存放路径-->
        keystoreType="PKCS12"   <!--这里是证书格式，.pfx和.p12都是PKCS12格式的-->
        keystorePass="123456"    <!--刚才输入的密码-->
   />
```

### 3、重启 Tomcat 服务器，访问测试
