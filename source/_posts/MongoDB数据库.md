---
title: MongoDB数据库
tags:
  - Linux
  - 技巧
  - MongDB
categories: 实训日志
declare: true
cover: https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220106012400.jpg
abbrlink: bd0448f7
date: 2020-06-18 09:51:35
---

## **前言**

MongoDB数据库的学习

<!-- more -->

## 一、MongoDB的简介

### 1.1 MongoDB的概述

![](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/img/mongo.jpg)

* MongoDB 是由C++语言编写的，是一个基于分布式文件存储的开源数据库系统。
* 在高负载的情况下，添加更多的节点，可以保证服务器性能。
* MongoDB 旨在为WEB应用提供可扩展的高性能数据存储解决方案。
* MongoDB 将数据存储为一个文档，数据结构由键值(key=>value)对组成
* MongoDB 文档类似于 JSON 对象。字段值可以包含其他文档，数组及文档数组。

## 二、MongoDB的安装

### 2.1 安装

`apt install mongodb`

安装完成可以用`mongo -version`来查看版本检查是否安装成功

![](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/img/mongo1.jpg)

### 2.2 MongoDB基本管理

通过以下命令，可以对 mongoDB 数据库进行一些基本的操作：

* 查看服务状态
  service mongodb status
* 启动服务
  service mongodb start
* 停止服务
  service mongodb stop
* 重新载入资源
  service mongodb reload

### 2.3 卸载

`apt --purge remove mongodb mongodb-clients mongodb-server`

## 三、MongoDB基本配置

### 3.1 MongoDB文件结构

mongoDB 数据库安装好以后，有以下四个比较重要的文件和目录：

* 主启动文件：/usr/bin/mongod  
* 配置文件：/etc/mongodb.conf  
* 日志文件存放目录：/var/log/mongodb  
* 数据存放位置目录：/var/lib/mongodb  

## 四、MongoDB基本操作

启动MongoDB服务后可以进行MongoDB的基本操作

### 4.1 连接数据库

使用`mongo`命令连接数据库：

![](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/img/mongo2.jpg)

### 4.2 查看已有的数据库

使用`show dbs`命令进行查看：

```
> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
> 
```

### 4.3 使用数据库

如果使用的数据库不存在，就创建同名数据库：

```
> use books
switched to db books  //选中books数据库
```

### 4.4 创建数据集合

类似于一张表，命令为`db.createCollection("表名")`

```
> db.createCollection("user")
{ "ok" : 1 }   //user创建成功
```

### 4.5 对数据集合进行操作

#### 4.5.1 插入数据

插入命令有两种

`db.表名.insert({})`

`db.表名.save({})`

两者区别在于：插入数据时，如果_id存在，insert操作时，则插入失败，save操作时，则更新数据

##### 插入一条数据：

```
> db.user.insert({_id:1,uname:"zhangsan",age:20,sex:"男"})  
WriteResult({ "nInserted" : 1 })   //一般情况，第一个字段都是_id,如果没有，也会自动添加一个_id
```

##### 插入多条数据：

```
> db.user.insert([{_id:2,uname:"lisi",age:19,sex:"男"},{_id:3,uname:"wangwu",age:18,sex:"女"}])   //插入多条数据中间用逗号分开，然后用[]包起来就行
BulkWriteResult({
        "writeErrors" : [ ],
        "writeConcernErrors" : [ ],
        "nInserted" : 2,
        "nUpserted" : 0,
        "nMatched" : 0,
        "nModified" : 0,
        "nRemoved" : 0,
        "upserted" : [ ]
})   
```

#### 4.5.2 查询数据

##### 不带条件的查询

查询命令：`db.表名.find()`

```
> db.user.find()
{ "_id" : 1, "uname" : "zhangsan", "age" : 20, "sex" : "男" }
{ "_id" : 2, "uname" : "lisi", "age" : 19, "sex" : "男" }
{ "_id" : 3, "uname" : "wangwu", "age" : 18, "sex" : "女" }
```

##### 带条件的查询

查询命令：`db.表名.find({条件})`

```
> db.user.find({sex:"男",uname:"zhangsan"})  //支持多条件查询
{ "_id" : 1, "uname" : "zhangsan", "age" : 20, "sex" : "男" }
```

#### 4.5.3 更新数据

更新命令：`db.表名.update({条件},{$set:{}})`  //这里条件也和查询一样支持多条件

```
> db.user.update({_id:1},{$set:{uname:"ouwen"}})  //把_id为1的uname改为ouwen
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.user.find()
{ "_id" : 1, "uname" : "ouwen", "age" : 20, "sex" : "男" }  //修改成功
{ "_id" : 2, "uname" : "lisi", "age" : 19, "sex" : "男" }
{ "_id" : 3, "uname" : "wangwu", "age" : 18, "sex" : "女" }
```

#### 4.5.4 删除数据

删除命令：`db.表名.remove({条件})`

```
> db.user.remove({uname:"lisi"}) //删除uname为lisi的数据
WriteResult({ "nRemoved" : 1 })
> db.user.find()
{ "_id" : 1, "uname" : "ouwen", "age" : 20, "sex" : "男" }
{ "_id" : 3, "uname" : "wangwu", "age" : 18, "sex" : "女" }
```

#### 4.5.5 查看所有的数据集

相当于查看表：`show collections`

```
show collections
user    //我这里只有一张表
```

#### 4.5.6 删除数据集合

类似于删除表：`db.表名.drop()`

```
> db.user.drop()
true    //成功删除user表
```

### 4.6 删除数据库

先使用要删除的数据库：`use books`

删除命令：`db.dropDatabase()`

```
> use books   //使用books数据库
switched to db books
> db.dropDatabase()   //删除它
{ "dropped" : "books", "ok" : 1 }
> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
```

### 4.7 退出数据库连接

可以使用`Crl+C`或者输入`exit`回车

```
> exit
bye
root@iZ2ze4ojx7qtz1wv44gajnZ:~# 
```

