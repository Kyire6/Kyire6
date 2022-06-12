---
title: SpringCloud快速搭建
tags:
  - 笔记
  - 技巧
  - 分布式
categories: Spring
cover: 'https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20211212182044.png'
abbrlink: 939440f7
date: 2021-06-22 16:25:44
updated: 2021-06-22 16:26:26
---

# SpringCloud快速搭建

## SpringCloud五大组件

- 服务注册与发现——Netflix Eureka
- 负载均衡：
  - 客户端负载均衡——Netflix Ribbon
  - 服务端负载均衡：——Feign(其也是依赖于Ribbon，只是将调用方式RestTemplete 更改成Service 接口)
- 断路器——Netflix Hystrix
- 服务网关——Netflix Zuul
- 分布式配置——Spring Cloud Config

## 常见面试题

-  什么是微服务？
-  微服务之间是如何独立通讯的？
-  SpringCloud 和 Dubbo有那些区别？
-  SpringBoot 和 SpringCloud，请谈谈你对他们的理解
-  什么是服务熔断？什么是服务降级？
-  微服务的优缺点分别是什么？说下你在项目开发中遇到的坑
-  你所知道的微服务技术栈有哪些？列举一二
-  Eureka和Zookeeper都可以提供服务注册与发现的功能，请说说两者的区别

## 微服务概述

### 什么是微服务

> 什么是微服务？

微服务(Microservice Architecture) 是近几年流行的一种架构思想，关于它的概念很难一言以蔽之。

究竟什么是微服务呢？我们在此引用ThoughtWorks 公司的首席科学家 Martin Fowler 于2014年提出的一段话：

原文：https://martinfowler.com/articles/microservices.html

汉化：https://www.cnblogs.com/liuning8023/p/4493156.html

- 就目前而言，对于微服务，业界并没有一个统一的，标准的定义。
- 但通常而言，微服务架构是一种架构模式，或者说是一种架构风格，**它体长将单一的应用程序划分成一组小的服务**，每个服务运行在其独立的自己的进程内，服务之间互相协调，互相配置，为用户提供最终价值，服务之间采用轻量级的通信机制(HTTP)互相沟通，每个服务都围绕着具体的业务进行构建，并且能狗被独立的部署到生产环境中，另外，应尽量避免统一的，集中式的服务管理机制，对具体的一个服务而言，应该根据业务上下文，选择合适的语言，工具(**Maven**)对其进行构建，可以有一个非常轻量级的集中式管理来协调这些服务，可以使用不同的语言来编写服务，也可以使用不同的数据存储。

> 再来从技术维度角度理解下：

- 微服务化的核心就是将传统的一站式应用，根据业务拆分成一个一个的服务，彻底地去耦合，每一个微服务提供单个业务功能的服务，一个服务做一件事情，从技术角度看就是一种小而独立的处理过程，类似进程的概念，能够自行单独启动或销毁，拥有自己独立的数据库。

### 微服务与微服务架构

> 微服务

强调的是服务的大小，它关注的是某一个点，是具体解决某一个问题/提供落地对应服务的一个服务应用，狭义的看，可以看作是IDEA中的一个个微服务工程，或者Moudel。IDEA 工具里面使用Maven开发的一个个独立的小Moudel，它具体是使用SpringBoot开发的一个小模块，专业的事情交给专业的模块来做，一个模块就做着一件事情。强调的是一个个的个体，每个个体完成一个具体的任务或者功能。

> 微服务架构

一种新的架构形式，Martin Fowler 于2014年提出。

微服务架构是一种架构模式，它体长将单一应用程序划分成一组小的服务，服务之间相互协调，互相配合，为用户提供最终价值。每个服务运行在其独立的进程中，服务与服务之间采用轻量级的通信机制**(如HTTP)互相协作，每个服务都围绕着具体的业务进行构建，并且能够被独立的部署到生产环境中，另外，应尽量避免统一的，集中式的服务管理机制，对具体的一个服务而言，应根据业务上下文，选择合适的语言、工具(如Maven)**对其进行构建。

### 微服务优缺点

> 优点

- 单一职责原则；
- 每个服务足够内聚，足够小，代码容易理解，这样能聚焦一个指定的业务功能或业务需求；
- 开发简单，开发效率高，一个服务可能就是专一的只干一件事；
- 微服务能够被小团队单独开发，这个团队只需2-5个开发人员组成；
- 微服务是松耦合的，是有功能意义的服务，无论是在开发阶段或部署阶段都是独立的；
- 微服务能使用不同的语言开发；
- 易于和第三方集成，微服务允许容易且灵活的方式集成自动部署，通过持续集成工具，如jenkins，Hudson，bamboo；
- 微服务易于被一个开发人员理解，修改和维护，这样小团队能够更关注自己的工作成果，无需通过合作才能体现价值；
- 微服务允许利用和融合最新技术；
- 微服务只是业务逻辑的代码，不会和HTML，CSS，或其他的界面混合;
- 每个微服务都有自己的存储能力，可以有自己的数据库，也可以有统一的数据库；

> 缺点

- 开发人员要处理分布式系统的复杂性；
- 多服务运维难度，随着服务的增加，运维的压力也在增大；
- 系统部署依赖问题；
- 服务间通信成本问题；
- 数据一致性问题；
- 系统集成测试问题；
- 性能和监控问题；

### 微服务技术栈

| **微服务技术条目**                     | 落地技术                                                     |
| -------------------------------------- | ------------------------------------------------------------ |
| 服务开发                               | SpringBoot、Spring、SpringMVC等                              |
| 服务配置与管理                         | Netfix公司的Archaius、阿里的Diamond等                        |
| 服务注册与发现                         | Eureka、Consul、Zookeeper等                                  |
| 服务调用                               | Rest、PRC、gRPC                                              |
| 服务熔断器                             | Hystrix、Envoy等                                             |
| 负载均衡                               | Ribbon、Nginx等                                              |
| 服务接口调用(客户端调用服务的简化工具) | Fegin等                                                      |
| 消息队列                               | Kafka、RabbitMQ、ActiveMQ等                                  |
| 服务配置中心管理                       | SpringCloudConfig、Chef等                                    |
| 服务路由(API网关)                      | Zuul等                                                       |
| 服务监控                               | Zabbix、Nagios、Metrics、Specatator等                        |
| 全链路追踪                             | Zipkin、Brave、Dapper等                                      |
| 数据流操作开发包                       | SpringCloud Stream(封装与Redis，Rabbit，Kafka等发送接收消息) |
| 时间消息总栈                           | SpringCloud Bus                                              |
| 服务部署                               | Docker、OpenStack、Kubernetes等                              |

### 为什么选择SpringCloud作为微服务架构

1. **选型依据**

   - 整体解决方案和框架成熟度
   - 社区热度
   - 可维护性
   - 学习曲线

2. **当前各大IT公司用的微服务架构有那些？**

   - 阿里：dubbo+HFS
   - 京东：JFS
   - 新浪：Motan
   - 当当网：DubboX

3. 各微服务框架对比

   ![image-20210424223407910](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210424223415.png)

## SpringCloud入门概述

### SpringCloud是什么？

Spring官网：https://spring.io/

![img](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210424223656.png)

![img](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210424223702.png)	

------

- Spring Cloud是一个微服务框架，相比Dubbo等RPC框架, **Spring Cloud提供的全套的分布式系统解决方案**。
- Spring Cloud对微服务基础框架Netflix的多个开源组件进行了封装，同时又实现了和云端平台以及和Spring Boot开发框架的集成。 
- Spring Cloud为微服务架构开发涉及的**配置管理，服务治理，熔断机制，智能路由，微代理，控制总线，一次性token，全局一致性锁，leader选举，分布式session，集群状态**管理等操作提供了一种简单的开发方式。
- Spring Cloud 为开发者提供了快速构建**分布式系统的工具**，开发者可以快速的启动服务或构建应用、同时能够快速和云平台资源进行对接。  

### SpringCloud和SpringBoot的关系

- SpringBoot专注于开苏方便的开发单个个体微服务；
- SpringCloud是关注全局的微服务协调整理治理框架，它将SpringBoot开发的一个个单体微服务，整合并管理起来，为各个微服务之间提供：配置管理、服务发现、断路器、路由、为代理、事件总栈、全局锁、决策竞选、分布式会话等等集成服务；
- SpringBoot可以离开SpringCloud独立使用，开发项目，但SpringCloud离不开SpringBoot，属于依赖关系；
- **SpringBoot专注于快速、方便的开发单个个体微服务，SpringCloud关注全局的服务治理框架；**

### Dubbo和SpringCloud技术选型

##### 分布式+服务治理Dubbo

目前成熟的互联网架构，应用服务化拆分 + 消息中间件

##### Dubbo 和 SpringCloud对比

最大区别：Spring Cloud 抛弃了Dubbo的RPC通信，采用的是基于HTTP的REST方式

严格来说，这两种方式各有优劣。虽然从一定程度上来说，后者牺牲了服务调用的性能，但也避免了上面提到的原生RPC带来的问题。而且REST相比RPC更为灵活，服务提供方和调用方的依赖只依靠一纸契约，不存在代码级别的强依赖，这个优点在当下强调快速演化的微服务环境下，显得更加合适。

### SpringCloud能干嘛？

- Distributed/versioned configuration 分布式/版本控制配置
- Service registration and discovery 服务注册与发现
- Routing 路由
- Service-to-service calls 服务到服务的调用
- Load balancing 负载均衡配置
- Circuit Breakers 断路器
- Distributed messaging 分布式消息管理

**自学参考书：**

- SpringCloud Netflix 中文文档：https://springcloud.cc/spring-cloud-netflix.html
- SpringCloud 中文API文档(官方文档翻译版)：https://springcloud.cc/spring-cloud-dalston.html
- SpringCloud中国社区：http://springcloud.cn/
- SpringCloud中文网：https://springcloud.cc

## Eureka服务注册与发现

### 什么是Eureka？

- Netflix在涉及Eureka时，遵循的就是API原则.
- Eureka是Netflix的有个子模块，也是核心模块之一。Eureka是基于REST的服务，用于定位服务，以实现云端中间件层服务发现和故障转移，服务注册与发现对于微服务来说是非常重要的，有了服务注册与发现，只需要使用服务的标识符，就可以访问到服务，而不需要修改服务调用的配置文件了，功能类似于Dubbo的注册中心，比如Zookeeper.

### 原理理解

- Eureka基本的架构
  - Springcloud 封装了Netflix公司开发的Eureka模块来实现服务注册与发现 (对比Zookeeper).
  - Eureka采用了C-S的架构设计，EurekaServer作为服务注册功能的服务器，他是服务注册中心.
  - 而系统中的其他微服务，使用Eureka的客户端连接到EurekaServer并维持心跳连接。这样系统的维护人员就可以通过EurekaServer来监控系统中各个微服务是否正常运行，Springcloud 的一些其他模块 (比如Zuul) 就可以通过EurekaServer来发现系统中的其他微服务，并执行相关的逻辑.

![image-20210516114206565](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210516114346.png)

- 和Dubbo架构对比.

![img](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210426230548.png)

- Eureka 包含两个组件：Eureka Server 和 Eureka Client.


- Eureka Server 提供服务注册，各个节点启动后，会在EurekaServer中进行注册，这样Eureka Server中的服务注册表中将会储存所有可用服务节点的信息，服务节点的信息可以在界面中直观的看到.


- Eureka Client 是一个Java客户端，用于简化EurekaServer的交互，客户端同时也具备一个内置的，使用轮询负载算法的负载均衡器。在应用启动后，将会向EurekaServer发送心跳 (默认周期为30秒) 。如果Eureka Server在多个心跳周期内没有接收到某个节点的心跳，EurekaServer将会从服务注册表中把这个服务节点移除掉 (默认周期为90s).

- **三大角色**
  - Eureka Server：提供服务的注册与发现
  - Service Provider：服务生产方，将自身服务注册到Eureka中，从而使服务消费方能狗找到
  - Service Consumer：服务消费方，从Eureka中获取注册服务列表，从而找到消费服务

### 构建Eureka

#### eureka-server

1. 新建springcloud-eureka-7001模块

2. pom.xml配置

   ```xml
   <!--导包~-->
   <dependencies>
       <!--导入Eureka Server依赖-->
       <dependency>
           <groupId>org.springframework.cloud</groupId>
           <artifactId>spring-cloud-starter-eureka-server</artifactId>
           <version>1.4.6.RELEASE</version>
       </dependency>
       <!--热部署工具-->
       <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-devtools</artifactId>
       </dependency>
   </dependencies>
   ```

3. application.yml

   ```yaml
   server:
     port: 7001
   eureka:
     instance:
       hostname: localhost #eureka服务端的实例名称
     client:
       register-with-eureka: false #是否向eureka注册中心注册自己
       fetch-registry: false # false代表自己为注册中心
       service-url: # 监控页面
         defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
   ```

4. 主启动类

   ```java
   package com.luojunjie.springcloud;
   
   import org.springframework.boot.SpringApplication;
   import org.springframework.boot.autoconfigure.SpringBootApplication;
   import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;
   
   /**
    * 启动之后，访问http://127.0.0.1/
    * @author IRVING
    * @create 2021-04-26 23:24
    */
   @SpringBootApplication
   @EnableEurekaServer //服务端的启动类，可以接受别人注册进来
   public class EurekaServer_7001 {
       public static void main(String[] args) {
           SpringApplication.run(EurekaServer_7001.class,args);
       }
   }
   ```

5. 启动成功后访问 http://localhost:7001/ 得到以下页面

   ![image-20210427094515719](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210427094523.png)

#### eureka-client

**只需要在之前的springcloud-provider-dept-8001**

1. 导入Eureka依赖

   ```xml
   <!-- Eureka -->
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-eureka</artifactId>
       <version>1.4.6.RELEASE</version>
   </dependency>
   ```

2. application中新增Eureka配置

   ```yaml
   # Eureka配置：配置服务注册中心地址
   eureka:
     client:
       service-url:
         defaultZone: http://localhost:7001/eureka/
     instance:
       instance-id: springcloud-provider-dept-8001 #修改eureka上的默认描述
   ```

3. 为主启动类添加@EnableEurekaClient注解

   ```java
   /**
    * @author IRVING
    * @create 2021-04-25 0:13
    */
   @SpringBootApplication
   @EnableEurekaClient //在服务启动后自动注册到eureka中
   public class DeptProvider_8001 {
       public static void main(String[] args) {
           SpringApplication.run(DeptProvider_8001.class, args);
       }
   }
   ```

4. 先启动7001服务端后启动8001客户端进行测试，然后访问监控页http://localhost:7001/ 查看结果如图，成功

   ![image-20210427100904350](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210427100932.png)

5. 配置关于服务加载的监控信息

   ![image-20210427101125312](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210427101125.png)

6. 在pom.xml文件中添加依赖

   ```xml
   <!--actuator完善监控信息-->
   <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
   </dependency>
   ```

7. application.yml中添加配置

   ```yaml
   # info配置
   info:
     # 项目配置
     app:
       #项目名称
       name: springcloud-dept8001
       #公司名称
       company: www.ljjblog.com
   ```

8. 刷新监控页，进入服务

   <img src="https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210427103412.png" alt="image-20210427103412175" style="zoom:50%;" />	

9. 跳转页面如下所示

   ![image-20210427103451458](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210427103451.png)

### Eureka自我保护机制：好死不如赖活着

一句话总结就是：**某时刻某一个微服务不可用，eureka不会立即清理，依旧会对该微服务的信息进行保存！**

- 默认情况下，当eureka server在一定时间内没有收到实例的心跳，便会把该实例从注册表中删除**（默认是90秒）**，但是，如果短时间内丢失大量的实例心跳，便会触发eureka server的自我保护机制，比如在开发测试时，需要频繁地重启微服务实例，但是我们很少会把eureka server一起重启（因为在开发过程中不会修改eureka注册中心），**当一分钟内收到的心跳数大量减少时，会触发该保护机制**。可以在eureka管理界面看到Renews threshold和Renews(last min)，当后者（最后一分钟收到的心跳数）小于前者（心跳阈值）的时候，触发保护机制，会出现红色的警告：==EMERGENCY!EUREKA MAY BE INCORRECTLY CLAIMING INSTANCES ARE UP WHEN THEY'RE NOT.RENEWALS ARE LESSER THAN THRESHOLD AND HENCE THE INSTANCES ARE NOT BEGING EXPIRED JUST TO BE SAFE==.从警告中可以看到，eureka认为虽然收不到实例的心跳，但它认为实例还是健康的，eureka会保护这些实例，不会把它们从注册表中删掉。
- 该保护机制的目的是避免网络连接故障，在发生网络故障时，微服务和注册中心之间无法正常通信，但服务本身是健康的，不应该注销该服务，如果eureka因网络故障而把微服务误删了，那即使网络恢复了，该微服务也不会重新注册到eureka server了，因为只有在微服务启动的时候才会发起注册请求，后面只会发送心跳和服务列表请求，这样的话，该实例虽然是运行着，但永远不会被其它服务所感知。所以，eureka server在短时间内丢失过多的客户端心跳时，会进入自我保护模式，该模式下，eureka会保护注册表中的信息，不在注销任何微服务，当网络故障恢复后，eureka会自动退出保护模式。自我保护模式可以让集群更加健壮。
- 但是我们在开发测试阶段，需要频繁地重启发布，如果触发了保护机制，则旧的服务实例没有被删除，这时请求有可能跑到旧的实例中，而该实例已经关闭了，这就导致请求错误，影响开发测试。所以，在开发测试阶段，我们可以把自我保护模式关闭，只需在eureka server配置文件中加上如下配置即可：==eureka.server.enable-self-preservation=false==【**不推荐关闭自我保护机制**】

### Eureka集群搭建

<img src="https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210427143231.png" alt="image-20210427143231543" style="zoom:50%;" />

#### 1、初始化

新建springcloud-eureka-7002、springcloud-eureka-7003模块

1.为pmx.xml添加依赖（与springcloud-eureka-7001相同）

```xml
<!-- 导包 -->
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-eureka-server</artifactId>
        <version>1.4.6.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
    </dependency>
</dependencies>
```

2.为application.yml配置（与springcloud-eureka-7001相同）

```yaml
server:
  port: 7002
eureka:
  instance:
    hostname: localhost #eureka服务端的实例名称
  client:
    register-with-eureka: false #是否向eureka注册中心注册自己
    fetch-registry: false # false代表自己为注册中心
    service-url: # 监控页面
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
```

3.主启动类(与springcloud-eureka-7001相同)

```java
/**
 * 启动之后，访问http://127.0.0.1/
 * @author IRVING
 * @create 2021-04-26 23:24
 */
@SpringBootApplication
@EnableEurekaServer //服务端的启动类，可以接受别人注册进来
public class EurekaServer_7002 {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServer_7002.class,args);
    }
}
```

#### 2、集群成员之间互相关联

配置一些自定义本机名字，找到本机hosts文件并打开

![image-20210427145825690](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210427145825.png)	

在hosts文件最后加上，要访问的本机名称，默认是localhost

![image-20210427145918816](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210427145918.png)	

在集群中使springcloud-eureka-7001关联springcloud-eureka-7002、springcloud-eureka-7003

完整的springcloud-eureka-7001下的application.yml如下：

```yaml
server:
  port: 7001
eureka:
  instance:
    hostname: eureka7001.com #eureka服务端的实例名称
  client:
    register-with-eureka: false #是否向eureka注册中心注册自己
    fetch-registry: false # false代表自己为注册中心
    service-url: # 监控页面
      #重写Eureka的默认端口以及访问路径 --->http://localhost:7001/eureka/
      # 单机： defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
      # 集群（关联）：7001关联7002、7003
      defaultZone: http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/
```

在集群中使springcloud-eureka-7002关联springcloud-eureka-7001、springcloud-eureka-7003

完整的springcloud-eureka-7002下的application.yml如下：

```yaml
server:
  port: 7002
eureka:
  instance:
    hostname: eureka7002.com #eureka服务端的实例名称
  client:
    register-with-eureka: false #是否向eureka注册中心注册自己
    fetch-registry: false # false代表自己为注册中心
    service-url: # 监控页面
      #重写Eureka的默认端口以及访问路径 --->http://localhost:7001/eureka/
      # 单机： defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
      # 集群（关联）：7001关联7002、7003
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7003.com:7003/eureka/
```

在集群中使springcloud-eureka-7003关联springcloud-eureka-7001、springcloud-eureka-7002

完整的springcloud-eureka-7003下的application.yml如下：

```yaml
server:
  port: 7003
eureka:
  instance:
    hostname: eureka7003.com #eureka服务端的实例名称
  client:
    register-with-eureka: false #是否向eureka注册中心注册自己
    fetch-registry: false # false代表自己为注册中心
    service-url: # 监控页面
      #重写Eureka的默认端口以及访问路径 --->http://localhost:7001/eureka/
      # 单机： defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
      # 集群（关联）：7001关联7002、7003
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/
```

通过springcloud-provider-dept-8001下的yml配置文件，修改**Eureka配置：配置服务注册中心地址**

```yaml
#Eureka配置，服务注册到哪里
eureka:
  client:
    service-url:
      #启用eureka集群配置
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/
  instance:
    instance-id: springcloud-provider-dept-8001 #修改eureka上的默认描述
```

这样模拟集群就搭建好了，就可以把一个项目挂载到三个服务器上了

![image-20210427151353122](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210427151353.png)

### 对比和Zookeeper区别

1. **回顾CAP原则**

   RDBMS (MySQL\Oracle\sqlServer) ===> ACID

   NoSQL (Redis\MongoDB) ===> CAP

2. **ACID是什么？**

   - A (Atomicity) 原子性
   - C (Consistency) 一致性
   - I (Isolation) 隔离性
   - D (Durability) 持久性

3. **CAP是什么？**

   - C (Consistency) 强一致性
   - A (Availability) 可用性
   - P (Partition tolerance) 分区容错性

   CAP的三进二：CA、AP、CP

4. ==CAP理论的核心==

   一个分布式系统不可能同时很好的满足一致性，可用性和分区容错性这三个需求

   根据CAP原理，将NoSQL数据库分成了满足CA原则，满足CP原则和满足AP原则三大类

   - CA：单点集群，满足一致性，可用性的系统，通常可扩展性较差
   - CP：满足一致性，分区容错的系统，通常性能不是特别高
   - AP：满足可用性，分区容错的系统，通常可能对一致性要求低一些

5. **作为分布式服务注册中心，Eureka比Zookeeper好在哪里？**

   著名的CAP理论指出，一个分布式系统不可能同时满足C (一致性) 、A (可用性) 、P (容错性)，由于分区容错性P在分布式系统中是必须要保证的，因此我们只能再A和C之间进行权衡。

   - Zookeeper 保证的是 CP —> 满足一致性，分区容错的系统，通常性能不是特别高
   - Eureka 保证的是 AP —> 满足可用性，分区容错的系统，通常可能对一致性要求低一些

   **Zookeeper保证的是CP**

    当向注册中心查询服务列表时，我们可以容忍注册中心返回的是几分钟以前的注册信息，但不能接收服务直接down掉不可用。也就是说，**服务注册功能对可用性的要求要高于一致性**。但zookeeper会出现这样一种情况，当master节点因为网络故障与其他节点失去联系时，剩余节点会重新进行leader选举。问题在于，选举leader的时间太长，30-120s，且选举期间整个zookeeper集群是不可用的，这就导致在选举期间注册服务瘫痪。在云部署的环境下，因为网络问题使得zookeeper集群失去master节点是较大概率发生的事件，虽然服务最终能够恢复，但是，漫长的选举时间导致注册长期不可用，是不可容忍的。

   **Eureka保证的是AP**

    Eureka看明白了这一点，因此在设计时就优先保证可用性。**Eureka各个节点都是平等的**，几个节点挂掉不会影响正常节点的工作，剩余的节点依然可以提供注册和查询服务。而Eureka的客户端在向某个Eureka注册时，如果发现连接失败，则会自动切换至其他节点，只要有一台Eureka还在，就能保住注册服务的可用性，只不过查到的信息可能不是最新的，除此之外，Eureka还有之中自我保护机制，如果在15分钟内超过85%的节点都没有正常的心跳，那么Eureka就认为客户端与注册中心出现了网络故障，此时会出现以下几种情况：

   - Eureka不在从注册列表中移除因为长时间没收到心跳而应该过期的服务
   - Eureka仍然能够接受新服务的注册和查询请求，但是不会被同步到其他节点上 (即保证当前节点依然可用)
   - 当网络稳定时，当前实例新的注册信息会被同步到其他节点中

   ==因此，Eureka可以很好的应对因网络故障导致部分节点失去联系的情况，而不会像zookeeper那样使整个注册服务瘫痪==

## Ribbon：负载均衡(基于客户端)

#### 负载均衡以及Ribbon

> **ribbon是什么？**

- Spring Cloud Ribbon是基于Netflix Ribbon实现的一套==客户端负载均衡工具==。
- 简单的说，Ribbon是Netflix发布的一个开源项目，主要功能是提供客户端的软件负载均衡算法，将Netflix的中间层服务连接在一起。Ribbon的客户端组件提供一系列完整的配置项如：连接超时、重试等等。简单的说，就是在配置文件中列出LoadBalancer（简称LB：负载均衡）后面所有的机器，Ribbon会自动的帮助你基于某种规则（如简单轮询、随机连接等等）去连接这些机器。我们也很容易使用Ribbon实现自定义的负载均衡算法！

> **ribbon能干嘛？**

- LB，即负载均衡 (LoadBalancer) ，在微服务或分布式集群中经常用的一种应用。
- 负载均衡简单的说就是将用户的请求平摊的分配到多个服务上，从而达到系统的HA (高用)。
- 常见的负载均衡软件有 Nginx、Lvs 等等。
- Dubbo、SpringCloud 中均给我们提供了负载均衡，**SpringCloud 的负载均衡算法可以自定义**。
- 负载均衡简单分类：
  - 集中式LB
    - 即在服务的提供方和消费方之间使用独立的LB设施，如Nginx(反向代理服务器)，由该设施负责把访问请求通过某种策略转发至服务的提供方！
  - 进程式 LB
    - 将LB逻辑集成到消费方，消费方从服务注册中心获知有哪些地址可用，然后自己再从这些地址中选出一个合适的服务器。
    - ==Ribbon 就属于进程内LB==，它只是一个类库，集成于消费方进程，消费方通过它来获取到服务提供方的地址！

#### SpringCloud集成Ribbon

向**springcloud-consumer-dept-80**模块的pom.xml中添加Ribbon和Eureka依赖

```xml
<!-- Ribbon+Eureka -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
    <version>1.4.6.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka</artifactId>
    <version>1.4.6.RELEASE</version>
</dependency>
```

在application.yml文件中配置Eureka

```yaml
#Eureka配置
eureka:
  client:
    register-with-eureka: false #不向注册中心注册自己
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/
```

主启动类加上@EnableEurekaClient注解，开启Eureka

```java
/**
 * Ribbon 和 Eureka 整合以后，客户端可以直接调用，不用关心IP地址和端口号
 * @author IRVING
 * @create 2021-04-25 0:36
 */
@SpringBootApplication
@EnableEurekaClient
public class DeptConsumer_80 {
    public static void main(String[] args) {
        SpringApplication.run(DeptConsumer_80.class);
    }
}
```

自定义Spring配置类：ConfigBean.java 配置负载均衡实现RestTemplate

```java
/**
 * @author IRVING
 * @create 2021-04-25 0:27
 */
@Configuration
public class ConfigBean {

    //配置负载均衡实现RestTemplate
    @Bean
    @LoadBalanced //Ribbon
    public RestTemplate getRestTemplate() {
        return new RestTemplate();
    }
}
```

#### 使用Ribbon实现负载均衡

![img](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210428011632.png)

1. 新建两个服务提供者Moudle：springcloud-provider-dept-8003、springcloud-provider-dept-8002
2. 参照springcloud-provider-dept-8001 依次为另外两个Moudle添加pom.xml依赖 、resourece下的mybatis和application.yml配置，Java代码
3. 启动所有服务测试(根据自身电脑配置决定启动服务的个数)，访问http://eureka7001.com:7002/查看结果

![image-20210428011818529](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210428011818.png)

测试访问http://localhost/consumer/dept/list 这时候随机访问的是服务提供者8001

<img src="https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210428012211.png" alt="image-20210428012211706" style="zoom: 50%;" />	

再次访问http://localhost/consumer/dept/list这时候随机的是服务提供者8002

<img src="https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210428012255.png" alt="image-20210428012255782" style="zoom: 50%;" />	

以上这种**每次访问http://localhost/consumer/dept/list随机访问集群中某个服务提供者，这种情况叫做轮询**，轮询算法在SpringCloud中可以自定义。

**如何切换或者自定义规则呢？**

在springcloud-provider-dept-80模块下的ConfigBean中进行配置，切换使用不同的规则

```java
@Configuration
public class ConfigBean {

    //配置负载均衡实现RestTemplate
    @Bean
    @LoadBalanced //Ribbon
    public RestTemplate getRestTemplate() {
        return new RestTemplate();
    }

    @Bean
    public IRule myRule() {
        return new RandomRule();//使用随机策略
        //return new RoundRobinRule();//使用轮询策略
        //return new AvailabilityFilteringRule();//使用轮询策略
        //return new RetryRule();//使用轮询策略
    }
}
```

也可以自定义规则，在myRule包下自定义一个配置类MyRule.java，注意：**该包不要和主启动类所在的包同级，要跟启动类所在包同级**：

<img src="https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210428014625.png" alt="image-20210428014625684" style="zoom:50%;" />	

**MyRule.java**

```java
@Configuration
public class MyRule {

    @Bean
    public IRule myRule(){
        return new MyRandomRule();//默认是轮询RandomRule,现在自定义为自己的
    }
}
```

主启动类开启负载均衡并指定自定义的MyRule配置类

```java
@SpringBootApplication
@EnableEurekaClient
//在微服务启动的时候就能加载自定义的Ribbon类(自定义的规则会覆盖原有默认的规则)
@RibbonClient(name = "springcloud-provider-dept",configuration = MyRandomRule.class)
public class DeptConsumer_80 {
    public static void main(String[] args) {
        SpringApplication.run(DeptConsumer_80.class);
    }
}
```

自定义的规则(这里我们参考Ribbon中默认的规则代码自己稍微改动)：MyRandomRule.java

```java
package com.luojunjie.myrule;

import com.netflix.client.config.IClientConfig;
import com.netflix.loadbalancer.AbstractLoadBalancerRule;
import com.netflix.loadbalancer.ILoadBalancer;
import com.netflix.loadbalancer.Server;

import java.util.List;
import java.util.concurrent.ThreadLocalRandom;

/**
 * @author IRVING
 * @create 2021-04-28 1:47
 */
public class MyRandomRule extends AbstractLoadBalancerRule {

    /**
     * 每个服务访问5次则换下一个服务(总共3个服务)
     * <p>
     * total=0,默认=0,如果=5,指向下一个服务节点
     * index=0,默认=0,如果total=5,index+1
     */
    private int total = 0;//被调用的次数
    private int currentIndex = 0;//当前是谁在提供服务

    //@edu.umd.cs.findbugs.annotations.SuppressWarnings(value = "RCN_REDUNDANT_NULLCHECK_OF_NULL_VALUE")
    public Server choose(ILoadBalancer lb, Object key) {
        if (lb == null) {
            return null;
        }
        Server server = null;

        while (server == null) {
            if (Thread.interrupted()) {
                return null;
            }
            List<Server> upList = lb.getReachableServers();//获得当前活着的服务
            List<Server> allList = lb.getAllServers();//获取所有的服务

            int serverCount = allList.size();
            if (serverCount == 0) {
                /*
                 * No servers. End regardless of pass, because subsequent passes
                 * only get more restrictive.
                 */
                return null;
            }

            //int index = chooseRandomInt(serverCount);//生成区间随机数
            //server = upList.get(index);//从或活着的服务中,随机获取一个

            //=====================自定义代码=========================

            if (total < 5) {
                server = upList.get(currentIndex);
                total++;
            } else {
                total = 0;
                currentIndex++;
                if (currentIndex > upList.size()) {
                    currentIndex = 0;
                }
                server = upList.get(currentIndex);//从活着的服务中,获取指定的服务来进行操作
            }

            //======================================================

            if (server == null) {
                /*
                 * The only time this should happen is if the server list were
                 * somehow trimmed. This is a transient condition. Retry after
                 * yielding.
                 */
                Thread.yield();
                continue;
            }
            if (server.isAlive()) {
                return (server);
            }
            // Shouldn't actually happen.. but must be transient or a bug.
            server = null;
            Thread.yield();
        }
        return server;
    }

    protected int chooseRandomInt(int serverCount) {
        return ThreadLocalRandom.current().nextInt(serverCount);
    }

    @Override
    public Server choose(Object key) {
        return choose(getLoadBalancer(), key);
    }

    @Override
    public void initWithNiwsConfig(IClientConfig clientConfig) {
        // TODO Auto-generated method stub
    }
}
```

## Feign：负载均衡(基于服务端)

### Feign简介

Feign是声明式Web Service客户端，它让微服务之间的调用变得更简单，==类似controller调用service==。SpringCloud集成了Ribbon和Eureka，可以使用Feigin提供负载均衡的http客户端

**只需要创建一个接口，然后添加注解即可~**

Feign，主要是社区版，大家都习惯面向接口编程。这个是很多开发人员的规范。调用微服务访问两种方法

- 微服务名字 【ribbon】
- 接口和注解 【feign】

**Feign能干什么？**

- Feign旨在使编写Java Http客户端变得更容易
- 前面在使用Ribbon + RestTemplate时，利用RestTemplate对Http请求的封装处理，形成了一套模板化的调用方法。但是在实际开发中，由于对服务依赖的调用可能不止一处，往往一个接口会被多处调用，所以通常都会针对每个微服务自行封装一个客户端类来包装这些依赖服务的调用。所以，Feign在此基础上做了进一步的封装，由他来帮助我们定义和实现依赖服务接口的定义，==在Feign的实现下，我们只需要创建一个接口并使用注解的方式来配置它 (类似以前Dao接口上标注Mapper注解，现在是一个微服务接口上面标注一个Feign注解)==，即可完成对服务提供方的接口绑定，简化了使用Spring Cloud Ribbon 时，自动封装服务调用客户端的开发量。

**Feign默认集成了Ribbon**

- 利用Ribbon维护了MicroServiceCloud-Dept的服务列表信息，并且通过轮询实现了客户端的负载均衡，而与Ribbon不同的是，通过Feign只需要定义服务绑定接口且以声明式的方法，优雅而简单的实现了服务调用。

### Feign的使用步骤

1. 创建springcloud-consumer-fdept-feign模块

   <img src="https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210428120942.png" alt="image-20210428120935729" style="zoom:50%;" />	

   拷贝springcloud-consumer-dept-80模块下的pom.xml，resource，以及java代码到springcloud-consumer-feign模块，并添加feign依赖。

   ```xml
   <!--Feign的依赖-->
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-feign</artifactId>
       <version>1.4.6.RELEASE</version>
   </dependency>
   ```

   通过**Ribbon**实现：—原来的controller：**DeptConsumerController.java**

   ```java
   package com.luojunjie.springcloud.controller;
   
   import com.luojunjie.springcloud.pojo.Dept;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.web.bind.annotation.PathVariable;
   import org.springframework.web.bind.annotation.RequestBody;
   import org.springframework.web.bind.annotation.RequestMapping;
   import org.springframework.web.bind.annotation.RestController;
   import org.springframework.web.client.RestTemplate;
   
   import java.util.List;
   
   /**
    * @author IRVING
    * @create 2021-04-25 0:24
    */
   @RestController
   public class DeptConsumerController {
       //消费者不应该有service层
       // RestTemplate 供我们直接调用! 注册到spring中
       // 提供多种便捷远程访问http服务的方法，简单的restful服务模板
   
       @Autowired
       private RestTemplate restTemplate;
   
       //通过Ribbon负载均衡，我们这里的地址应该是一个变量，通过服务名来访问
       //private static final String REST_URL_PREFIX = "http://localhost:8001";
       private static final String REST_URL_PREFIX = "http://springcloud-provider-dept";
   
       //http://localhost:8001/dept/get/1
       @RequestMapping("/consumer/dept/get/{id}")
       public Dept get(@PathVariable("id") Long id) {
           return restTemplate.getForObject(REST_URL_PREFIX + "/dept/get/" + id, Dept.class);
       }
   
       @RequestMapping("/consumer/dept/add")
       public boolean add(@RequestBody Dept dept) {
           return restTemplate.postForObject(REST_URL_PREFIX + "/dept/add/", dept, boolean.class);
       }
   
       @RequestMapping("/consumer/dept/list")
       public List<Dept> list() {
           return restTemplate.getForObject(REST_URL_PREFIX + "/dept/list", List.class);
       }
   }
   ```

   通过**Feign**实现：—改造后controller：**DeptConsumerController.java**

   ```java
   package com.luojunjie.springcloud.controller;
   
   import com.luojunjie.springcloud.pojo.Dept;
   import com.luojunjie.springcloud.service.DeptClientService;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.beans.factory.annotation.Qualifier;
   import org.springframework.web.bind.annotation.PathVariable;
   import org.springframework.web.bind.annotation.RequestBody;
   import org.springframework.web.bind.annotation.RequestMapping;
   import org.springframework.web.bind.annotation.RestController;
   
   import java.util.List;
   
   /**
    * @author IRVING
    * @create 2021-04-25 0:24
    */
   @RestController
   public class DeptConsumerController {
   
       @Autowired
       private DeptClientService service = null;
   
       @RequestMapping("/consumer/dept/get/{id}")
       public Dept get(@PathVariable("id") Long id) {
           return service.queryById(id);
       }
   
       @RequestMapping("/consumer/dept/add")
       public boolean add(@RequestBody Dept dept) {
           return service.addDept(dept);
       }
   
       @RequestMapping("/consumer/dept/list")
       public List<Dept> list() {
           return service.queryAll();
       }
   }
   ```

   Feign和Ribbon二者对比，前者显现出面向接口编程特点，代码看起来更清爽，而且==Feign调用方式更符合我们之前在做SSM或者SprngBoot项目时，Controller层调用Service层的编程习惯==！

   主配置类：

   ```java
   package com.luojunjie.springcloud;
   
   import org.springframework.boot.SpringApplication;
   import org.springframework.boot.autoconfigure.SpringBootApplication;
   import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
   import org.springframework.cloud.openfeign.EnableFeignClients;
   
   /**
    * Ribbon 和 Eureka 整合以后，客户端可以直接调用，不用关心IP地址和端口号
    * @author IRVING
    * @create 2021-04-25 0:36
    */
   @SpringBootApplication
   @EnableEurekaClient
   @EnableFeignClients(basePackages = {"com.luojunjie.springcloud"})
   public class FeignDeptConsumer_80 {
       public static void main(String[] args) {
           SpringApplication.run(FeignDeptConsumer_80.class);
       }
   }
   ```

2. 改造springcloud-api模块

   pom.xml添加feign依赖

   ```xml
   <!-- https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-feign -->
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-feign</artifactId>
       <version>1.4.6.RELEASE</version>
   </dependency>
   ```

   新建service包，并新建DeptClientService.java接口

   ```java
   /**
    * @author IRVING
    * @create 2021-04-28 11:37
    */
   @Component
   @FeignClient(value = "springcloud-provider-dept")
   public interface DeptClientService {
   
       @GetMapping("/dept/get/{id}")
       Dept queryById(@PathVariable("id") Long id);
   
       @GetMapping("/dept/list")
       List<Dept> queryAll();
   
       @PostMapping("/dept/add")
       boolean addDept(Dept dept);
   }
   
   ```

## Hystrix：服务熔断

> 分布式系统面临的问题

==复杂的分布式体系结构中的应用程序中有数十个依赖关系，每个依赖关系在某些时候将不可避免失败！==

### 服务雪崩

多个微服务之间调用的时候，假设微服务A调用微服务B和微服务C，微服务B和微服务C又调用其他的微服务，这就是所谓的“扇出”，如果扇出的链路上**某个微服务的调用响应时间过长，或者不可用**，对微服务A的调用就会占用越来越多的系统资源，进而引起系统崩溃，所谓的“雪崩效应”。

![img](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210428151122.png)	

对于高流量的应用来说，单一的后端依赖可能会导致所有服务器上的所有资源都在几十秒内饱和。比失败更糟糕的是，这些应用程序还可能导致服务之间的延迟增加，备份队列，线程和其他系统资源紧张，导致整个系统发生更多的级联故障，**这些都表示需要对故障和延迟进行隔离和管理，以达到单个依赖关系的失败而不影响整个应用程序或系统运行**。

**我们需要，弃车保帅！**

### 什么是Hystrix？

**Hystrix**是一个应用于处理分布式系统的延迟和容错的开源库，在分布式系统里，许多依赖不可避免的会调用失败，比如超时，异常等，**Hystrix**==能够保证在一个依赖出问题的情况下，不会导致整个体系服务失败，避免级联故障，以提高分布式系统的弹性==。

 “**断路器**”本身是一种开关装置，当某个服务单元发生故障之后，通过断路器的故障监控 (类似熔断保险丝) ，**向调用方返回一个服务预期的，可处理的备选响应 (FallBack) ，而不是长时间的等待或者抛出调用方法无法处理的异常，这样就可以保证了服务调用方的线程不会被长时间，不必要的占用**，从而避免了故障在分布式系统中的蔓延，乃至雪崩。

![img](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210428151551.png)	

### Hystrix能干嘛？

- 服务降级
- 服务熔断
- 服务限流
- 接近实时的监控

当一切正常时，请求流可以如下所示：

![img](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210428151740.png)	

当许多后端系统中有一个潜在阻塞服务时，它可以阻止整个用户请求：

![img](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210428151740.png)	

随着大容量通信量的增加，单个后端依赖项的潜在性会导致所有服务器上的所有资源在几秒钟内饱和。

应用程序中通过网络或客户端库可能导致网络请求的每个点都是潜在故障的来源。比失败更糟糕的是，这些应用程序还可能导致服务之间的延迟增加，从而备份队列、线程和其他系统资源，从而导致更多跨系统的级联故障。

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9naXRodWIuY29tL05ldGZsaXgvSHlzdHJpeC93aWtpL2ltYWdlcy9zb2EtMy02NDAucG5n?x-oss-process=image/format,png)	

当使用**Hystrix**包装每个基础依赖项时，上面的图表中所示的体系结构会发生类似于以下关系图的变化。**每个依赖项是相互隔离的**，限制在延迟发生时它可以填充的资源中，并包含在回退逻辑中，该逻辑决定在依赖项中发生任何类型的故障时要做出什么样的响应：

![在这里插入图片描述](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210428152041.png)	

### 服务熔断

**什么是服务熔断?**

**熔断机制是赌赢雪崩效应的一种微服务链路保护机制。**

当扇出链路的某个微服务不可用或者响应时间太长时，会进行服务的降级，**进而熔断该节点微服务的调用，快速返回错误的响应信息**。检测到该节点微服务调用响应正常后恢复调用链路。在SpringCloud框架里熔断机制通过Hystrix实现。Hystrix会监控微服务间调用的状况，当失败的调用到一定阀值缺省是**5秒内20次调用失败，就会启动熔断机制**。熔断机制的注解是：@HystrixCommand。

服务熔断解决如下问题：

- 当所依赖的对象不稳定时，能够起到快速失败的目的；
- 快速失败后，能够根据一定的算法动态试探所依赖对象是否恢复。

**入门案例**

新建springcloud-provider-dept-hystrix-8001模块并拷贝springcloud-provider-dept–8001内的pom.xml、resource和Java代码进行初始化并调整。

**导入hystrix依赖**

```xml
<!--导入Hystrix依赖-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-hystrix</artifactId>
    <version>1.4.6.RELEASE</version>
</dependency>
```

**调整yml配置文件**

```yaml
server:
  port: 8001
#mybatis配置
mybatis:
  type-aliases-package: com.luojunjie.springcloud.pojo
  mapper-locations: classpath:mybatis/mapper/*.xml
  config-location: classpath:mybatis/mybatis-config.xml
#spring配置
spring:
  application:
    name: springcloud-provider-dept
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/db01?useSSL=false&serverTimezone=Asia/Shanghai&allowPublicKeyRetrieval=true
    username: root
    password: root
#Eureka配置，服务注册到哪里
eureka:
  client:
    service-url:
      #启用eureka集群配置
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/
  instance:
    instance-id: springcloud-provider-dept-hystrix-8001 #修改eureka上的默认描述
    prefer-ip-address: true #改为true后默认显示的是ip地址而不再是localhost
# info配置
info:
  # 项目配置
  app:
    #项目名称
    name: springcloud-dept8001
    #公司名称
    company: www.ljjblog.com
```

**修改controller**

```java
package com.luojunjie.springcloud.controller;

import com.luojunjie.springcloud.pojo.Dept;
import com.luojunjie.springcloud.service.DeptService;
import com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

import java.util.List;

/**
 * @author IRVING
 * @create 2021-04-25 0:09
 */
@RestController
public class DeptController {

    @Autowired
    private DeptService deptService;

    /**
     * 根据id查询部门信息
     * 如果根据id查询出现异常,则走hystrixGet这段备选代码
     *
     * @param id
     * @return
     */
    @GetMapping("/dept/get/{id}")
    @HystrixCommand(fallbackMethod = "hystrixGet")
    public Dept get(@PathVariable("id") Long id) {
        Dept dept = deptService.queryById(id);
        if (dept==null){
            throw new RuntimeException("这个id=>"+id+",不存在该用户，或信息无法找到~");
        }
        return dept;
    }


    /**
     * 根据id查询备选方案(熔断)
     *
     * @param id
     * @return
     */
    public Dept hystrixGet(@PathVariable("id") Long id) {
        return new Dept().setDeptno(id)
                .setDname("这个id=>" + id + ",没有对应的信息,null---@Hystrix~")
                .setDb_source("在MySQL中没有这个数据库");
    }

}
```

**为主启动类添加对熔断的支持注解@EnableCircuitBreaker**

```java
package com.luojunjie.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.circuitbreaker.EnableCircuitBreaker;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

/**
 * 在服务启动后自动注册到eureka中
 *
 * @author IRVING
 * @create 2021-04-25 0:13
 */
@SpringBootApplication
@EnableDiscoveryClient
@EnableCircuitBreaker //添加对熔断的支持注解
public class HystrixDeptProvider_8001 {
    public static void main(String[] args) {
        SpringApplication.run(HystrixDeptProvider_8001.class, args);
    }
}
```

**测试**：

使用熔断后，当访问一个不存在的id时，前台页展示数据如下:

![image-20210428224831260](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210428224838.png)

**为了避免因某个微服务后台出现异常或错误而导致整个应用或网页报错，使用熔断是必要的**

### 服务降级

什么是服务降级?

服务降级是指 当服务器压力剧增的情况下，根据实际业务情况及流量，对一些服务和页面有策略的不处理，或换种简单的方式处理，从而释放服务器资源以保证核心业务正常运作或高效运作。说白了，**就是尽可能的把系统资源让给优先级高的服务**。

资源有限，而请求是无限的。如果在并发高峰期，不做服务降级处理，一方面肯定会影响整体服务的性能，严重的话可能会导致宕机某些重要的服务不可用。所以，一般在高峰期，为了保证核心功能服务的可用性，都要对某些服务降级处理。比如当双11活动时，把交易无关的服务统统降级，如查看蚂蚁深林，查看历史订单等等。

服务降级主要用于什么场景呢？当整个微服务架构整体的负载超出了预设的上限阈值或即将到来的流量预计将会超过预设的阈值时，为了保证重要或基本的服务能正常运行，可以将一些 不重要 或 不紧急 的服务或任务进行服务的 延迟使用 或 暂停使用。

降级的方式可以根据业务来，可以延迟服务，比如延迟给用户增加积分，只是放到一个缓存中，等服务平稳之后再执行 ；或者在粒度范围内关闭服务，比如关闭相关文章的推荐。

![在这里插入图片描述](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210428230041.png)

由上图可得，**当某一时间内服务A的访问量暴增，而B和C的访问量较少，为了缓解A服务的压力，这时候需要B和C暂时关闭一些服务功能，去承担A的部分服务，从而为A分担压力，叫做服务降级**。

**服务降级需要考虑的问题**

1）那些服务是核心服务，哪些服务是非核心服务
2）那些服务可以支持降级，那些服务不能支持降级，降级策略是什么
3）除服务降级之外是否存在更复杂的业务放通场景，策略是什么？

**自动降级分类**

1）超时降级：主要配置好超时时间和超时重试次数和机制，并使用异步机制探测回复情况

2）失败次数降级：主要是一些不稳定的api，当失败调用次数达到一定阀值自动降级，同样要使用异步机制探测回复情况

3）故障降级：比如要调用的远程服务挂掉了（网络故障、DNS故障、http服务返回错误的状态码、rpc服务抛出异常），则可以直接降级。降级后的处理方案有：默认值（比如库存服务挂了，返回默认现货）、兜底数据（比如广告挂了，返回提前准备好的一些静态页面）、缓存（之前暂存的一些缓存数据）

4）限流降级：秒杀或者抢购一些限购商品时，此时可能会因为访问量太大而导致系统崩溃，此时会使用限流来进行限制访问量，当达到限流阀值，后续请求会被降级；降级后的处理方案可以是：排队页面（将用户导流到排队页面等一会重试）、无货（直接告知用户没货了）、错误页（如活动太火爆了，稍后重试）。

##### 入门案例

在springcloud-api模块下的service包中新建降级配置类DeptClientServiceFallBackFactory.java

```java
package com.luojunjie.springcloud.service;

import com.luojunjie.springcloud.pojo.Dept;
import feign.hystrix.FallbackFactory;

import java.util.List;

/**
 * @author IRVING
 * @create 2021-04-28 23:03
 */
public class DeptClientServiceFallBackFactory implements FallbackFactory {
    @Override
    public Object create(Throwable throwable) {
        return new DeptClientService() {
            @Override
            public Dept queryById(Long id) {
                return new Dept()
                        .setDeptno(id)
                        .setDname("id=>" + id + "没有对应的信息，客户端提供了降级的信息，这个服务现在已经被关闭")
                        .setDb_source("没有数据~");
            }

            @Override
            public List<Dept> queryAll() {
                return null;
            }

            @Override
            public boolean addDept(Dept dept) {
                return false;
            }
        };
    }
}
```

在DeptClientService中指定降级配置类DeptClientServiceFallBackFactory

```java
package com.luojunjie.springcloud.service;

import com.luojunjie.springcloud.pojo.Dept;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.stereotype.Component;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;

import java.util.List;

/**
 * @author IRVING
 * @create 2021-04-28 11:37
 */
@Component
@FeignClient(value = "springcloud-provider-dept",fallbackFactory = DeptClientServiceFallBackFactory.class)
public interface DeptClientService {

    @GetMapping("/dept/get/{id}")
    Dept queryById(@PathVariable("id") Long id);

    @GetMapping("/dept/list")
    List<Dept> queryAll();

    @PostMapping("/dept/add")
    boolean addDept(Dept dept);
}
```

在**springcloud-consumer-dept-feign**模块中开启降级：

```yaml
server:
  port: 80
#Eureka配置
eureka:
  client:
    register-with-eureka: false #不向注册中心注册自己
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/
#开启降级feign.hystrix
feign:
  hystrix:
    enabled: true
```

#### 服务熔断和服务降级的区别

- 服务熔断—>服务端：某个服务超时或异常，引起熔断~，类似于保险丝(自我熔断)
- 服务降级—>客户端：从整体网站请求负载考虑，当某个服务熔断或者关闭之后，服务将不再被调用，此时在客户端，我们可以准备一个 FallBackFactory ，返回一个默认的值(缺省值)。会导致整体的服务下降，但是好歹能用，比直接挂掉强。
- 触发原因不太一样，服务熔断一般是某个服务（下游服务）故障引起，而服务降级一般是从整体负荷考虑；管理目标的层次不太一样，熔断其实是一个框架级的处理，每个微服务都需要（无层级之分），而降级一般需要对业务有层级之分（比如降级一般是从最外围服务开始）
- 实现方式不太一样，服务降级具有代码侵入性(由控制器完成/或自动降级)，熔断一般称为自我熔断。

**熔断，降级，限流：**

限流：限制并发的请求访问量，超过阈值则拒绝；

降级：服务分优先级，牺牲非核心服务（不可用），保证核心服务稳定；从整体负荷考虑；

熔断：依赖的下游服务故障触发熔断，避免引发本系统崩溃；系统自动执行和恢复

### Dashboard 流监控

新建springcloud-consumer-hystrix-dashboard模块

**添加依赖**

```xml
<dependencies>
    <!--导入Hystrix依赖-->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-hystrix</artifactId>
        <version>1.4.6.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-hystrix-dashboard</artifactId>
        <version>1.4.6.RELEASE</version>
    </dependency>
</dependencies>
```

**主启动类**

```java
@SpringBootApplication
@EnableHystrixDashboard
public class DeptConsumerDashboard_9001 {
    public static void main(String[] args) {
        SpringApplication.run(DeptConsumerDashboard_9001.class,args);
    }
}
```

给springcloud-provider-dept-hystrix-8001模块下的主启动类添加如下代码，开放监控流端口

```java
package com.luojunjie.springcloud;

import com.netflix.hystrix.contrib.metrics.eventstream.HystrixMetricsStreamServlet;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.web.servlet.ServletRegistrationBean;
import org.springframework.cloud.client.circuitbreaker.EnableCircuitBreaker;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.context.annotation.Bean;

/**
 * 在服务启动后自动注册到eureka中
 *
 * @author IRVING
 * @create 2021-04-25 0:13
 */
@SpringBootApplication
@EnableDiscoveryClient
@EnableCircuitBreaker //添加对熔断的支持注解
public class HystrixDeptProvider_8001 {
    public static void main(String[] args) {
        SpringApplication.run(HystrixDeptProvider_8001.class, args);
    }

    @Bean
    public ServletRegistrationBean hystrixMetricsStreamServlet(){
        ServletRegistrationBean registrationBean = new ServletRegistrationBean(new HystrixMetricsStreamServlet());
        //访问该页面就是监控页面
        registrationBean.addUrlMappings("/actuator/hystrix.stream");
        return registrationBean;
    }
}


```

访问：http://localhost:9001/hystrix

<img src="https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210429160335.png" alt="image-20210429160328204" style="zoom:50%;" />

进入监控页面，效果如下：

<img src="https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210429160510.png" alt="image-20210429160510275" style="zoom:50%;" />

![img](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210429160522.png)	

## Zuul路由网关

#### 概述

> 什么是Zuul？

Zuul包含了对请求的路由和过滤的两个最主要的功能：

其中路由功能负责将外部请求转发到具体的微服务实例上，是实现外部访问统一入口的基础，而过滤器功能则负责对请求的处理过程进行干预，是实现请求校验，服务聚合等功能的基础。Zuul和Eureka进行整合，将Zuul自身注册为Eureka中获得其他微服务的消息，也即以后的访问微服务都是通过Zuul跳转后获得。

- 注意：Zuul服务器最终还是会注册Eureka
- 提供：==代理 + 路由 + 过滤== 三大功能！

> Zuul能干嘛？

- 路由
- 过滤

**入门案例**

**新建springcloud-zuul模块，并导入依赖**

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-zuul</artifactId>
        <version>1.4.6.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-eureka</artifactId>
        <version>1.4.6.RELEASE</version>
    </dependency>
    <!--导入Hystrix依赖-->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-hystrix</artifactId>
        <version>1.4.6.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-hystrix-dashboard</artifactId>
        <version>1.4.6.RELEASE</version>
    </dependency>
</dependencies>
```

**application.yml**

```yaml
server:
  port: 9527
spring:
  application:
    name: springcloud-zuul
eureka:
  client:
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/
  instance:
    instance-id: zuul9527.com
    prefer-ip-address: true

info:
  app.name: luojunjie-springcloud
  company.name: luojunjie

zuul:
  routes:
    mydept.serviceId: springcloud-provider-dept
    mydept.path: /mydept/**
  ignored-services: "*" #不能使用这个路径访问了，* 忽略全部的
  prefix: /ouwen #公共的前缀
```

**主启动类**

```java
package com.luojunjie.spingcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.zuul.EnableZuulProxy;

/**
 * @author IRVING
 * @create 2021-04-29 21:55
 */
@SpringBootApplication
@EnableZuulProxy
public class ZuulApplication_9527 {
    public static void main(String[] args) {
        SpringApplication.run(ZuulApplication_9527.class,args);
    }
}
```

**测试：**

![image-20210429221139509](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210429221146.png)

可以看出Zull路由网关被注册到Eureka注册中心中了！

![image-20210429221300961](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210429221301.png)

## Spring Cloud Config

**Dalston.RELEASE**

**Spring Cloud Config为分布式系统中的外部配置提供服务器和客户端支持。**使用Config Server，您可以在所有环境中管理应用程序的外部属性。客户端和服务器上的概念映射与Spring `Environment`和`PropertySource`抽象相同，因此它们与Spring应用程序非常契合，但可以与任何以任何语言运行的应用程序一起使用。随着应用程序通过从开发人员到测试和生产的部署流程，您可以管理这些环境之间的配置，并确定应用程序具有迁移时需要运行的一切。服务器存储后端的默认实现使用git，因此它轻松支持标签版本的配置环境，以及可以访问用于管理内容的各种工具。很容易添加替代实现，并使用Spring配置将其插入。

#### 概述

**分布式系统面临的–配置文件问题**

微服务意味着要将单体应用中的业务拆分成一个个子服务，每个服务的粒度相对较小，因此系统中会出现大量的服务，由于每个服务都需要必要的配置信息才能运行，所以一套集中式的，动态的配置管理设施是必不可少的。spring cloud提供了configServer来解决这个问题，我们每一个微服务自己带着一个application.yml，那上百个的配置文件修改起来，令人头疼！

**什么是SpringCloud config分布式配置中心？**

![在这里插入图片描述](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210429225000.png)

spring cloud config 为微服务架构中的微服务提供集中化的外部支持，配置服务器为各个不同微服务应用的所有环节提供了一个**中心化的外部配置**。

spring cloud config 分为**服务端**和**客户端**两部分。

服务端也称为 **分布式配置中心**，它是一个独立的微服务应用，用来连接配置服务器并为客户端提供获取配置信息，加密，解密信息等访问接口。

客户端则是**通过指定的配置中心来管理应用资源，以及与业务相关的配置内容，并在启动的时候从配置中心获取和加载配置信息**。配置服务器默认采用git来存储配置信息，这样就有助于对环境配置进行版本管理。并且可用通过git客户端工具来方便的管理和访问配置内容。

**spring cloud config 分布式配置中心能干嘛？**

- 集中式管理配置文件
- 不同环境，不同配置，动态化的配置更新，分环境部署，比如 /dev /test /prod /beta /release
- 运行期间动态调整配置，不再需要在每个服务部署的机器上编写配置文件，服务会向配置中心统一拉取配置自己的信息
- 当配置发生变动时，服务不需要重启，即可感知到配置的变化，并应用新的配置
- 将配置信息以REST接口的形式暴露

**spring cloud config 分布式配置中心与GitHub整合**

由于spring cloud config 默认使用git来存储配置文件 (也有其他方式，比如自持SVN 和本地文件)，但是最推荐的还是git ，而且使用的是 http / https 访问的形式。

#### **入门案例**

**服务端**

新建springcloud-config-server-3344模块导入pom.xml依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-config-server</artifactId>
        <version>2.1.1.RELEASE</version>
    </dependency>
</dependencies>
```

resource下创建application.yml配置文件，Spring Cloud Config服务器从git存储库（必须提供）为远程客户端提供配置：

```yaml
server:
  port: 3344
spring:
  application:
    name: springcloud-config-server
     # 连接码云远程仓库
  cloud:
    config:
      server:
        git:
          uri: https://gitee.com/ouwen666/springcloud-config.git
```

主启动类

```java
package com.luojunjie.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.config.server.EnableConfigServer;

/**
 * @author IRVING
 * @create 2021-04-29 23:12
 */
@SpringBootApplication
@EnableConfigServer
public class SpringCloudConfigServer_3344 {
    public static void main(String[] args) {
        SpringApplication.run(SpringCloudConfigServer_3344.class, args);
    }
}
```

将本地git仓库springcloud-config文件夹下新建的application.yml提交到码云仓库：

![image-20210430000230137](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210430000230.png)

定位资源的默认策略是克隆一个git仓库（在`spring.cloud.config.server.git.uri`），并使用它来初始化一个迷你`SpringApplication`。小应用程序的`Environment`用于枚举属性源并通过JSON端点发布。

HTTP服务具有以下格式的资源：

```
/{application}/{profile}[/{label}]
/{application}-{profile}.yml
/{label}/{application}-{profile}.yml
/{application}-{profile}.properties
/{label}/{application}-{profile}.properties
```

其中“应用程序”作为SpringApplication中的spring.config.name注入（即常规的Spring Boot应用程序中通常是“应用程序”），“配置文件”是活动配置文件（或逗号分隔列表的属性），“label”是可选的git标签（默认为“master”）。

测试访问http://localhost:3344/application-dev.yml

![image-20210430000331332](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210430000331.png)

测试访问 http://localhost:3344/application/test/master

![image-20210430000351363](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210430000351.png)

测试访问 http://localhost:3344/master/application-dev.yml

![image-20210430000417885](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210430000417.png)

如果测试访问不存在的配置则不显示 如：http://localhost:3344/master/application-aaa.yml

![image-20210430000445929](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210430000446.png)

**客户端**

将本地git仓库springcloud-config文件夹下新建的config-client.yml提交到码云仓库：

![image-20210430000230137](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210430000526.png)

新建一个springcloud-config-client-3355模块，并导入依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-config</artifactId>
        <version>2.1.1.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

resources下创建application.yml和bootstrap.yml配置文件

**bootstrap.yml** 是系统级别的配置

```yaml
#系统级别的配置
spring:
  cloud:
    config:
      name: config-client #需要从git上读取的资源名称，不需要后缀
      uri: http://localhost:3344
      profile: dev
      label: master
```

**application.yml** 是用户级别的配置

```yaml
#用户级别的配置
spring:
  application:
    name: springcloud-config-client
```

创建controller包下的**ConfigClientController.java** 用于测试

```java
package com.luojunjie.springcloud.controller;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * @author IRVING
 * @create 2021-04-29 23:44
 */
@RestController
public class ConfigClientController {

    @Value("${spring.application.name}")
    private String applicationName; //获取微服务名称

    @Value("${eureka.client.service-url.defaultZone}")
    private String eurekaServer; //获取Eureka服务

    @Value("${server.port}")
    private String port; //获取服务端的端口号


    @RequestMapping("/config")
    public String getConfig(){
        return "applicationName:"+applicationName +
                "eurekaServer:"+eurekaServer +
                "port:"+port;
    }
}
```

主启动类

```java
package com.luojunjie.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

/**
 * @author IRVING
 * @create 2021-04-29 23:44
 */
@SpringBootApplication
public class SpringCloudConfigClient_3355 {
    public static void main(String[] args) {
        SpringApplication.run(SpringCloudConfigClient_3355.class,args);
    }
}
```

测试：

启动服务端Config_server_3344 再启动客户端ConfigClient

访问：http://localhost:8201/config/

![image-20210430000723324](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210430000723.png)
