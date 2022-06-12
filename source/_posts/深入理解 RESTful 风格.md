---
title: 深入理解RESTful风格
tags:
  - 技巧
  - 笔记
categories: Java
cover: https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220106010829.png
abbrlink: 6b25d652
date: 2021-06-12 19:24:44
updated: 2021-06-12 19:24:44
---
# 到底什么是 RESTful ？

## 前言

大家对 RESTful 肯定不陌生，就算不知道它到底是什么，但肯定听过这个玩意。相信大家也很想知道 RESTful 能解决什么样的问题，有什么应用场景？看完本篇文章你就能明白：

在互联网并没有盛行的时代， 移动端也还没有发展起来，页面请求和并发量也不高，那时候人们对接口的要求并不高，一些常规的动态页面（JSP）就能满足大部分人们的需求。

<img src="https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210610104619.png" alt="image-20210610104612710" style="zoom: 67%;" />

但是随着移动设备和互联网的发展，人们对 Web 应用的需求也逐渐增加，传统的动态页面（JSP）也因效率低下而渐渐被 HTML + JavaScript（Ajax）的前后端分离所替代，而安卓、IOS、小程序等不同的客户端层出不穷，客户端的种类出现多元化，**而客户端需要接口跟服务端进行通信**，但接口的 **规范性** 就成了一个问题：

<img src="https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210610105253.png" alt="image-20210610105253175" style="zoom: 67%;" />

所以一套 **结构清晰、符合标准、易于理解、扩展方便** 并且让大部分人都能理解并接收的接口风格就显得尤为重要，而 RESTful 风格的接口（RESTful API）刚好符合以上标准，就逐渐被应用从而流行起来。

<img src="https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210610105729.png" alt="image-20210610105729739" style="zoom: 67%;" />

现在，RESTful 是目前最流行的接口设计规范，在很多公司有着广泛的使用。在开发实践中我们很多人可能还是使用传统API进行请求交互，很多人其实并不特别了解RESTful API，对RESTful API的认知可能会停留在：

- 面向资源类型的
- 是一种风格
- （误区）接口传递参数使用斜杆（/）分割而用问号（?）传参

而其实一个很大的误区不要认为没有查询字符串就是RESTful API，也不要认为用了查询字符串就不是RESTful API，更不要认为用了JSON传输的API就是RESTful API。

## 一、REST 介绍

REST涉及一些概念性的东西可能比较多，在实战RESTful API之前，要对REST相关的知识有个系统的认知。

### REST 的诞生

REST（英文：Representational State Transfer，简称REST，直译过来表现层状态转换）是一种软件架构风格、设计风格，而不是标准，只是提供了一组设计原则和约束条件。它主要用于客户端和服务器交互类的软件。基于这个风格设计的软件可以更简洁，更有层次，更易于实现缓存等机制。

它首次出现在 2000 年 Roy Thomas Fielding 的博士论文中，这篇论文定义并详细介绍了表述性状态转移（Representational State Transfer，REST）的架构风格，并且描述了 如何使用 REST 来指导现代 Web 架构的设计和开发。用他自己的原话说：

> 写这篇文章的目的是：在符合架构原理前提下，理解和评估基于网络的应用软件的架构设计，得到一个功能强、性能好、适宜通信的架构。

需要注意的是 **REST并没有一个明确的标准，而更像是一种设计的风格** ，满足这种设计风格的程序或接口我们称之为RESTful(从单词字面来看就是一个形容词)。所以RESTful API 就是满足REST架构风格的接口。

### REST 架构特征

既然知道REST和RESTful的联系和区别，现在就要开始好好了解RESTful的一些约束条件和规则，RESTful是一种风格而不是标准，而这个风格大致有以下几个主要 **特征**：

**以资源为基础** ：资源可以是一个图片、音乐、一个XML格式、HTML格式或者JSON格式等网络上的一个实体，除了一些二进制的资源外普通的文本资源更多以JSON为载体、面向用户的一组数据(通常从数据库中查询而得到)。

**统一接口**: 对资源的操作包括获取、创建、修改和删除，这些操作正好对应HTTP协议提供的GET、POST、PUT和DELETE方法。换言而知，使用RESTful风格的接口但从接口上你可能只能定位其资源，但是无法知晓它具体进行了什么操作，需要具体了解其发生了什么操作动作要从其HTTP请求方法类型上进行判断。具体的HTTP方法和方法含义如下：

- GET（SELECT）：从服务器取出资源（一项或多项）。
- POST（CREATE）：在服务器新建一个资源。
- PUT（UPDATE）：在服务器更新资源（客户端提供完整资源数据）。
- PATCH（UPDATE）：在服务器更新资源（客户端提供需要修改的资源数据）。
- DELETE（DELETE）：从服务器删除资源。

当然也有很多在具体使用的时候使用PUT表示更新。从请求的流程来看，RESTful API和传统API大致架构如下：

<img src="https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210610133407.png" alt="image-20210610133407454" style="zoom:67%;" />

**URI指向资源**：URI = Universal Resource Identifier 统一资源标志符，用来标识抽象或物理资源的一个紧凑字符串。URI包括URL和URN，在这里更多时候可能代指URL(统一资源定位符)。RESTful是面向资源的，每种资源可能由一个或多个URI对应，但一个URI只指向一种资源。

**无状态**：服务器不能保存客户端的信息， 每一次从客户端发送的请求中，要包含所有必须的状态信息，会话信息由客户端保存， 服务器端根据这些状态信息来处理请求。当客户端可以切换到一个新状态的时候发送请求信息， 当一个或者多个请求被发送之后, 客户端就处于一个状态变迁过程中。每一个应用的状态描述可以被客户端用来初始化下一次的状态变迁。

### REST 架构限制条件

Fielding在论文中提出REST架构的6个**限制条件**，也可称为RESTful 6大原则， 标准的REST约束应满足以下6个原则：

**客户端-服务端（Client-Server）**: 这个更专注客户端和服务端的分离，服务端独立可更好服务于前端、安卓、IOS等客户端设备。

**无状态（Stateless）**：服务端不保存客户端状态，客户端保存状态信息每次请求携带状态信息。

**可缓存性（Cacheability）** ：服务端需回复是否可以缓存以让客户端甄别是否缓存提高效率。

**统一接口（Uniform Interface）**：通过一定原则设计接口降低耦合，简化系统架构，这是RESTful设计的基本出发点。

**分层系统（Layered System）**：客户端无法直接知道连接的到终端还是中间设备，分层允许你灵活的部署服务端项目。

**按需代码（Code-On-Demand，可选）**：按需代码允许我们灵活的发送一些看似特殊的代码给客户端例如JavaScript代码。

## 二、RESTful API 设计规范

既然了解了RESTful的一些规则和特性，那么具体该怎么去设计一个RESTful API呢？要从URL路径、HTTP请求动词、状态码和返回结果等方面详细考虑。

### URL 设计规范

URL为统一资源定位器 ,接口属于服务端资源，首先要通过URL定位到资源才能去访问，而通常一个完整的URL组成由以下几个部分构成：

```
URI = scheme "://" host  ":"  port "/" path [ "?" query ][ "#" fragment ]
```

- scheme: 指底层用的协议，如http、https、ftp
- host: 服务器的IP地址或者域名
- port: 端口，http默认为80端口
- path: 访问资源的路径，就是各种web 框架中定义的route路由
- query: 查询字符串，为发送给服务器的参数，在这里更多发送数据分页、排序等参数。
- fragment: 锚点，定位到页面的资源

我们在设计 API 时 URL 的 path 是需要认真考虑的，而 RESTful 对 path 的设计做了一些规范，通常一个 RESTful API 的 path 组成如下：

```
/{version}/{resources}/{resource_id}
```

- version：API版本号，有些版本号放置在头信息中也可以，通过控制版本号有利于应用迭代。
- resources：资源，RESTful API推荐用小写英文单词的复数形式。
- resource_id：资源的id，访问或操作该资源。

当然，有时候可能资源级别较大，其下还可细分很多子资源也可以灵活设计URL的path，例如：

```
/{version}/{resources}/{resource_id}/{subresources}/{subresource_id}
```

此外，有时可能增删改查无法满足业务要求，可以在URL末尾加上action，例如：

```
/{version}/{resources}/{resource_id}/action
```

其中 action 就是对资源的操作。

从大体样式了解URL路径组成之后，对于RESTful API的URL具体设计的规范如下：

1. 不用大写字母，所有单词使用英文且小写。
2. 连字符用中杠`"-"`而不用下杠`"_"`
3. 正确使用 `"/"`表示层级关系,URL的层级不要过深，并且越靠前的层级应该相对越稳定
4. 结尾不要包含正斜杠分隔符`"/"`
5. URL中不出现动词，用请求方式表示动作
6. 资源表示用复数不要用单数
7. 不要使用文件扩展名

### HTTP 动词

在 `RESTful API` 中，不同的HTTP请求方法有各自的含义，这里就展示 `GET,POST,PUT,DELETE` 几种请求API的设计与含义分析。针对不同操作，具体的含义如下：

```
GET /collection：从服务器查询资源的列表（数组）
GET /collection/resource：从服务器查询单个资源
POST /collection：在服务器创建新的资源
PUT /collection/resource：更新服务器资源
DELETE /collection/resource：从服务器删除资源
```

在非RESTful风格的API中，我们通常使用GET请求和POST请求完成增删改查以及其他操作，查询和删除一般使用GET方式请求，更新和插入一般使用POST请求。从请求方式上无法知道API具体是干嘛的，所有在URL上都会有操作的动词来表示API进行的动作，例如：`query，add，update，delete` 等等。

而 RESTful 风格的 API 则要求在 URL 上的都以名词的方式出现，从几种请求方式上就可以看出想要进行的操作，这与非 RSETful 风格的 API 形成鲜明的对比。

在谈及 `GET,POST,PUT,DELETE` 的时候，就必须提一下接口的 **安全性和幂等性**，其中安全性是指方法不会修改资源状态，即读的为安全的，写的操作为非安全的。而幂等性的意思是操作一次和操作多次的最终效果相同，客户端重复调用也只返回同一个结果。

上述四个HTTP请求方法的安全性和幂等性如下：

![image-20210610143431940](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210610143432.png)

### 状态码和返回数据

服务端处理完成后客户端也可能不知道具体成功了还是失败了，服务器响应时，包含**状态码**和**返回数据**两个部分。

#### 状态码

我们首先要正确使用各类状态码来表示该请求的处理执行结果。状态码主要分为五大类：

> 1xx：相关信息
> 2xx：操作成功
> 3xx：重定向
> 4xx：客户端错误
> 5xx：服务器错误

每一大类有若干小类，状态码的种类比较多，而主要常用状态码罗列在下面：

- 200 `OK - [GET]`：服务器成功返回用户请求的数据，该操作是幂等的（Idempotent）
- 201 `CREATED - [POST/PUT/PATCH]`：用户新建或修改数据成功
- 202 `Accepted - [*]`：表示一个请求已经进入后台排队（异步任务）
- 204 `NO CONTENT - [DELETE]`：用户删除数据成功。
- 400 `INVALID REQUEST - [POST/PUT/PATCH]`：用户发出的请求有错误，服务器没有进行新建或修改数据的操作，该操作是幂等的
- 401 `Unauthorized - [*]`：表示用户没有权限（令牌、用户名、密码错误）
- 403 `Forbidden - [*]` 表示用户得到授权（与401错误相对），但是访问是被禁止的
- 404 `NOT FOUND - [*]`：用户发出的请求针对的是不存在的记录，服务器没有进行操作，该操作是幂等的
- 406 `Not Acceptable - [GET]`：用户请求的格式不可得（比如用户请求JSON格式，但是只有XML格式）
- 410 `Gone -[GET]`：用户请求的资源被永久删除，且不会再得到的
- 422 `Unprocesable entity - [POST/PUT/PATCH]` 当创建一个对象时，发生一个验证错误
- 500 `INTERNAL SERVER ERROR - [*]`：服务器发生错误，用户将无法判断发出的请求是否成功

#### 返回结果

针对不同操作，服务器向用户返回数据，而各个团队或公司封装的返回实体类也不同，但都返回JSON格式数据给客户端。

## 总结

RESTful风格的API 固然很好很规范，但大多数互联网公司并没有按照或者完全按照其规则来设计，因为REST是一种风格，而不是一种约束或规则，过于理想的RESTful API 会付出太多的成本。

比如RESTful API也有一些缺点：

- 比如操作方式繁琐，`RESTful API` 通常根据 `GET、POST、PUT、DELETE` 来区分操作资源的动作，而 HTTP Method 本身不可直接见，是隐藏的，而如果将动作放到 URL 的 `path` 上反而清晰可见，更利于团队的理解和交流。
- 并且有些浏览器对 `GET,POST` 之外的请求支持不太友好，还需要特殊额外的处理。
- 过分强调资源，而实际业务API可能有各种需求比较复杂，单单使用资源的增删改查可能并不能有效满足使用需求，强行使用`RESTful` 风格API只会增加开发难度和成本。

所以，当你或你们的技术团队在设计 API 的时候，如果使用场景和 REST 风格很匹配，那么你们可以采用 RESTful 风格 API。但是如果业务需求和 RESTful 风格 API 不太匹配或者很麻烦，那也可以不用 RESTful 风格 API 或者可以借鉴一下，毕竟无论那种风格的 API 都是为了方便团队开发、协商以及管理，不能墨守成规。

