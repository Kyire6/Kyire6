---
title: RabbitMQ消息队列
tags:
  - 笔记
  - 技巧
  - RabbitMQ
categories: 中间件
cover: https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20211212181732.png
abbrlink: eb9166f8
date: 2021-04-19 16:25:44
updated: 2021-04-19 16:26:26
---
# RabbitMQ消息队列

## 1、什么是MQ

消息队列（Message Queue，简称MQ），从字面意思上看，本质是个队列，FIFO先入先出，只不过队列中存放的内容是message而已。其主要用途：不同进程Process/线程Thread之间通信。

**为什么会产生消息队列？有几个原因：**

- 不同进程（process）之间传递消息时，两个进程之间耦合程度过高，改动一个进程，引发必须修改另一个进程，为了隔离这两个进程，在两进程间抽离出一层（一个模块），所有两进程之间传递的消息，都必须通过消息队列来传递，单独修改某一个进程，不会影响另一个；
- 不同进程（process）之间传递消息时，为了实现标准化，将消息的格式规范化了，并且，某一个进程接受的消息太多，一下子无法处理完，并且也有先后顺序，必须对收到的消息进行排队，因此诞生了事实上的消息队列；

## 2、RabbitMQ

**RabbitMQ简介**

<img src="https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210414173139.png" alt="img"  />	

**开发语言：Erlang - 面向并发的编程语言**

<img src="https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210414173220.png" alt="img"  />	

**AMQP协议**

![img](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210414230450.png)	

**学习五种队列**

![img](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210414230702.png)

## 3、RabbitMQ 的第一个程序

### 第一种模型（直连）

<img src="https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210414235259.png" alt="image-20210414235259013" style="zoom:50%;" />	

- P：生产者：也就是要发送消息的程序
- C：消费者：消息的接受者，会一直等待消息的到来
- queue：消息队列，图中红色部分，类似一个邮箱，可以缓存消息；生产者向其中投递消息，消费者从其中取出消息

#### 建立一个maven项目

<img src="https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210414231107.png" alt="image-20210414231107272" style="zoom:50%;" />	

#### 导入RabbitMQ的客户端依赖

```xml
 <!-- 引入rabbitmq的相关依赖 -->
<dependency>
    <groupId>com.rabbitmq</groupId>
    <artifactId>amqp-client</artifactId>
    <version>5.7.2</version>
</dependency>
```

#### 编写生产者

```java
@Test
public void testSendMessage() throws IOException, TimeoutException {
    //创建连接mq的连接工厂对象
    ConnectionFactory connectionFactory = new ConnectionFactory();
    //设置连接rabbitmq主机
    connectionFactory.setHost("192.168.90.140");
    //设置端口号
    connectionFactory.setPort(5672);
    //设置连接哪个虚拟主机
    connectionFactory.setVirtualHost("/ems");
    //设置访问虚拟主机的用户名和密码
    connectionFactory.setUsername("ems");
    connectionFactory.setPassword("ems");

    //获取连接对象
    Connection connection = connectionFactory.newConnection();

    //获取连接中的通道对象
    Channel channel = connection.createChannel();

    //通道绑定对应的消息队列
    //参数一：队列名称 如果不存在自动创建
    //参数二：用来定义队列特性是否要持久化，true持久化队列 false不持久化
    //参数三：exclusive 是否独占队列 ture独占队列 false 不独占队列
    //参数四：autoDelete 是否在消费完成后自动删除队列 true 自动删除 false 不自动删除
    //参数五：额外参数
    channel.queueDeclare("hello",false,false,false,null);

    //发布消息
    //参数一：交换机名称 参数二：队列名称 参数三：传递消息额外名称 参数四：消息的具体内容
    channel.basicPublish("","hello",null,"hello rabbitmq".getBytes());

    channel.close();
    connection.close();
}
```

#### 编写消费者

```java
//创建连接工厂
ConnectionFactory connectionFactory = new ConnectionFactory();
connectionFactory.setHost("192.168.90.140");
connectionFactory.setPort(5672);
connectionFactory.setVirtualHost("/ems");
connectionFactory.setUsername("ems");
connectionFactory.setPassword("ems");

//创建连接对象
Connection connection = connectionFactory.newConnection();

//创建通道
Channel channel = connection.createChannel();

//通道绑定对象
channel.queueDeclare("hello", false, false, false, null);

//消费消息
//参数1；消费哪个队列的消息，队列名称
//参数2：开始消息的自动确认机制
//参数3：消费时的回调接口
channel.basicConsume("hello", true, new DefaultConsumer(channel) {
    @Override //最后一个参数：消息队列中取出的消息
    public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
        System.out.println("new String(body)==>" + new String(body));
    }
});
```

**注意：需要在rabbitmq管理页面中添加用户和虚拟主机**

![image-20210414231807927](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210414231808.png)

#### 编写连接工具类

```java
public class RabbitMQUtils {

    public static ConnectionFactory connectionFactory;

    static {
        //重量级资源 类加载的时候执行，只执行一次
        connectionFactory = new ConnectionFactory();
        connectionFactory.setHost("192.168.159.140");
        connectionFactory.setPort(5672);
        connectionFactory.setVirtualHost("/ems");
        connectionFactory.setUsername("ems");
        connectionFactory.setPassword("ems");
    }

    //定义提供连接的方法
    public static Connection getConnection() throws IOException, TimeoutException {
        try {
            return connectionFactory.newConnection();

        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }

    //定义关闭通道和关闭连接工具方法
    public static void closeConnectionAndChanel(Channel channel, Connection connection) {
        try {
            if (channel != null) {
                channel.close();
            }
            if (connection != null) {
                connection.close();
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

### 第二种模型（work queue）

`Work queue`，也被称为（`Task queue`），任务模型。当消息处理比较耗时的时候，可能生产消息的速度会远远大于消息的消费速度。长此以往，消息就会堆积越来越多，无法及时处理，此时就可以使用work 模型，让多个消费者绑定到一个队列，共同消费队列中的消息，队列中的消息一旦消费，就会消失，因此任务是不会被重复执行的。

![image-20210415000422705](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210415000422.png)	

角色：

- P：生产者：任务的发布者
- C1：消费者：领取任务并且完成任务，假设完成速度较慢
- C2：消费者2：领取任务并完成任务，假设完成速度快

#### 编写生产者

```java
//获取连接对象
Connection connection = RabbitMQUtils.getConnection();
//获取连接通道
Channel channel = connection.createChannel();

//通过通道声明队列
channel.queueDeclare("work", true, false, false, null);

for (int i = 0; i < 20; i++) {
    //生产消息
    channel.basicPublish("", "work", null, (i + "hello work queue").getBytes());
}

//关闭资源
RabbitMQUtils.closeConnectionAndChanel(channel, connection);
```

#### 编写消费者-1

```java
//获取连接
Connection connection = RabbitMQUtils.getConnection();
Channel channel = connection.createChannel();

channel.queueDeclare("work",true,false,false,null);

channel.basicConsume("work",true,new DefaultConsumer(channel){
    @Override
    public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties,byte[] body) throws IOException {
        System.out.println("消费者--1："+new String(body));
    }
});
```

#### 编写消费者-2

```java
//获取连接
Connection connection = RabbitMQUtils.getConnection();
Channel channel = connection.createChannel();

channel.queueDeclare("work",true,false,false,null);

channel.basicConsume("work",true,new DefaultConsumer(channel){
    @Override
    public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties,byte[] body) throws IOException {
        System.out.println("消费者--2："+new String(body));
    }
});
```

#### 测试结果

<img src="https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210415190334.png" alt="image-20210415190327510"  />

<img src="https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210415190348.png" alt="image-20210415190348026"  />					

==**总结：默认情况下，RabbitMQ将按顺序将每个消息发送给下一个使用者。平均而言，每个消费者都会收到相同数量的消息。这种分发消息的方式成为循环**==

### 消息自动确认机制

> 完成一项任务可能只需要几秒钟。您可能想知道，如果其中一个消费者启动了一个很长的任务，并且只完成了部分任务而死亡，会发生什么情况。在我们当前的代码中，一旦RabbitMQ向消费者发送消息，它就会立即标记该消息为删除。在本例中，如果您杀死一个worker，我们将丢失它正在处理的消息。我们还将丢失所有已发送到这个特定工作器但尚未处理的消息。

```java
//每一次只能消费一个消息
channel.basicQos(1);
//参数1：队列名称 参数2：消息自动确认 true 消费者自动向rabbitmq确认消息消费 false 不会自动确认
channel.basicConsume("work",false,new DefaultConsumer(channel){
	@Override
	public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
        System.out.println("消费者--1："+new String(body));
        // 参数1：确认队列中哪个具体消息 参数2：是否开启多个消息同时确认
        channel.basicAck(envelope.getDeliveryTag(),false);
    }
});
```

- 设置通道一次只能消费一个消息
- 关闭消息的自动确认，开启手动确认消息

![image-20210415193731262](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210415193731.png)

![image-20210415193743820](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210415193743.png)

### 第三种模型（fanout）

==`fanout` 扇出 也称为广播==

![image-20210415194629428](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210415194629.png)	

在广播模式下，消息发送流程是这样的：

- 可以有多个消费者
- 每个**消费者有自己的queue**（队列）
- 每个**队列都要绑定到Exchange**（交换机）
- **生产者发送的消息，只能发送到交换机**，交换机来决定要发给哪个队列，生产者无法决定
- 交换机把消息发送给绑定过的所有队列
- 队列的消费者都能拿到消息，实现一条消息被多个消费者消费

#### 编写生产者

```java
Connection connection = RabbitMQUtils.getConnection();
Channel channel = connection.createChannel();

//将通道声明指定交换机 参数1：交换机名称 参数2：交换机类型 fanout 广播类型
channel.exchangeDeclare("logs", "fanout");

//发送消息
channel.basicPublish("logs", "", null, "fanout type message".getBytes());

//释放资源
RabbitMQUtils.closeConnectionAndChanel(channel, connection);
```

#### 编写消费者-1

```java
//获取连接对象
Connection connection = RabbitMQUtils.getConnection();
Channel channel = connection.createChannel();

//通道绑定交换机
channel.exchangeDeclare("logs", "fanout");

//临时队列
String queue = channel.queueDeclare().getQueue();

//绑定交换机和队列
channel.queueBind(queue, "logs", "");

//消费消息
channel.basicConsume(queue, true, new DefaultConsumer(channel) {
    @Override
    public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties,byte[] body) throws IOException {
        System.out.println("消费者1==>" + new String(body));
    }
});
```

#### 编写消费者-2

```java
//获取连接对象
Connection connection = RabbitMQUtils.getConnection();
Channel channel = connection.createChannel();

//通道绑定交换机
channel.exchangeDeclare("logs", "fanout");

//临时队列
String queue = channel.queueDeclare().getQueue();

//绑定交换机和队列
channel.queueBind(queue, "logs", "");

//消费消息
channel.basicConsume(queue, true, new DefaultConsumer(channel) {
    @Override
    public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties,byte[] body) throws IOException {
        System.out.println("消费者2==>" + new String(body));
    }
});
```

### 第四种模型（Routing）

#### Routing 之订阅模型 -Direct（直连）

==在Fanout模式中，一条消息，会被所有订阅的队列消息。但是，在某些场景下，我们希望不同的消息被不同的队列消费。这时就要用到Direct类型的 Exchange。==

在Direct模型下：

- 队列与交换机的绑定，不能是任意绑定了，而是要指定一个 `RoutingKey`（路由key）
- 消息的发送方在向 Exchange发送消息是，也必须指定消息的 `RoutingKey`
- Exchange不再把消息交给每一个绑定的队列，而是根据消息的 `RoutingKey` 进行判断，只有队列的`RoutingKey` 与消息的 `RoutingKey` 完全一致，才会接收到消息

流程：

![image-20210415201126889](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210415201126.png)	

图解：

- P：生产者，向Exchange发送消息，发送消息是，会指定一个 Routing Key
- X：Exchange（交换机），接收生产者消息，然后把消息递交给与 Routing Key完全匹配的队列
- C1：消费者，其所在队列指定了需要Routing Key 为 error 的消息
- C2：消费者，其所在队列指定了需要Routing Key 为 info、 error、warning 的消息

##### 编写生产者

```java
Connection connection = RabbitMQUtils.getConnection();
Channel channel = connection.createChannel();

String exchangeName = "logs_direct";

//将通道声明指定交换机 参数1：交换机名称 参数2：交换机类型 direct 路由模式
channel.exchangeDeclare(exchangeName, "direct");

//发送消息
String routingKey = "info";
channel.basicPublish(exchangeName, routingKey, null, ("这是direct模型发布对的基于routing key["+routingKey+"]==>发送的消息").getBytes());

//释放资源
RabbitMQUtils.closeConnectionAndChanel(channel, connection);
```

##### 编写消费者-1

```java
//获取连接对象
Connection connection = RabbitMQUtils.getConnection();
Channel channel = connection.createChannel();

String exchangeName = "logs_direct";

//临时队列
String queue = channel.queueDeclare().getQueue();

//基于route key绑定队列和交换机
channel.queueBind(queue, exchangeName, "error");

//获取消费的消息
channel.basicConsume(queue, true, new DefaultConsumer(channel) {
    @Override
    public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties,byte[] body) throws IOException {
        System.out.println("消费者1==>" + new String(body));
    }
});
```

##### 编写消费者-2

```java
//获取连接对象
Connection connection = RabbitMQUtils.getConnection();
Channel channel = connection.createChannel();

String exchangeName = "logs_direct";

//临时队列
String queue = channel.queueDeclare().getQueue();

//绑定交换机和临时队列
channel.queueBind(queue, exchangeName, "info");
channel.queueBind(queue, exchangeName, "error");
channel.queueBind(queue, exchangeName, "warning");

//消费消息
channel.basicConsume(queue, true, new DefaultConsumer(channel) {
    @Override
    public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties,byte[] body) throws IOException {
        System.out.println("消费者2==>" + new String(body));
    }
});
```

#### Routing 之订阅模型 -Topic

`Topic` 类型的 `Exchange` 与 `Direct` 相比，都可以根据 `RoutingKey` 把消息路由到不用的队列。只不过 `Topic` 类型的 `Exchange` 可以让队列在绑定 `RoutingKey` 的时候使用通配符！这种模型 `RoutingKey` 一般都是由一个或多个单词组成，多个单词之间以“.”分割，例如： `item.insert`

![image-20210416092201943](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210416092208.png)	

##### 编写生产者

```java
//获取连接对象
Connection connection = RabbitMQUtils.getConnection();
Channel channel = connection.createChannel();

//声明交换机以及交换机类型 topic
channel.exchangeDeclare("topics", "topic");

//定义路由key
String routingKey = "user.save";
//发送消息
channel.basicPublish("topics", routingKey, null, ("这里是topic动态路由模型，routingKey：" + routingKey).getBytes());

//释放资源
RabbitMQUtils.closeConnectionAndChanel(channel, connection);
```

##### 编写消费者-1

```java
Connection connection = RabbitMQUtils.getConnection();
Channel channel = connection.createChannel();

//创建一个临时队列
String queue = channel.queueDeclare().getQueue();
//绑定队列和交换机，动态通配符形式routingKey
channel.queueBind(queue, "topics", "user.*");

//消费消息
channel.basicConsume(queue, true, new DefaultConsumer(channel) {
    @Override
    public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties,byte[] body) throws IOException {
        System.out.println("消费者1 ==>" + new String(body));
    }
});
```

##### 编写消费者-2

```java
Connection connection = RabbitMQUtils.getConnection();
Channel channel = connection.createChannel();

//创建一个临时队列
String queue = channel.queueDeclare().getQueue();
//绑定队列和交换机，动态通配符形式routingKey
channel.queueBind(queue, "topics", "user.#");

//消费消息
channel.basicConsume(queue, true, new DefaultConsumer(channel) {
    @Override
    public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
        System.out.println("消费者2 ==>" + new String(body));
    }
});
```

##### 结果：

![image-20210416094708394](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210416094708.png)

![image-20210416094717636](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210416094717.png)

## 4、SpringBoot 整合RabbitMQ

### 搭建初始环境

#### 1.引入依赖

```xml
<!-- 引入spring-rabbitmq依赖-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

#### 2.配置配置文件

```yaml
spring:
    application:
        name: rabbitmq-springboot
    rabbitmq:
        host: 192.168.80.140
        port: 5672
        username: ems
        password: ems
        virtual-host: /ems
```

==`RabbitTemplate` 用来简化操作  使用时候直接在项目中注入即可使用==

### HelloWorld 模型

#### 1.编写生产者

```java
//注入rabbitTemplate
@Autowired
private RabbitTemplate rabbitTemplate;

//hello world
@Test
public void testHello() {
    rabbitTemplate.convertAndSend("hello", "hello world");
}
```

#### 2.编写消费者

```java
@Component  //持久化 不独占 不是自动删除队列
@RabbitListener(queuesToDeclare = @Queue("hello"))
public class HelloConsumer {

    @RabbitHandler
    public void read(String message) {
        System.out.println("message==" + message);
    }
}
```

### Work 模型

#### 1.编写生产者

```java
//注入rabbitTemplate
@Autowired
private RabbitTemplate rabbitTemplate;

//work
@Test
public void testWork() {
    for (int i = 0; i < 10; i++) {
        rabbitTemplate.convertAndSend("work", "work模型" + i);
    }
}
```

#### 2.编写消费者

```java
@Component
public class WorkConsumer {
    //一个消费者
    @RabbitListener(queuesToDeclare = @Queue("work"))
    public void read1(String message) {
        System.out.println("message1=" + message);
    }

    //二个消费者
    @RabbitListener(queuesToDeclare = @Queue("work"))
    public void read2(String message) {
        System.out.println("message2=" + message);
    }
}
```

==**说明：默认在Spring AMQP实现中Work这种方式就是公平调度，如果需要实现能者多劳需要额外配置**==

### Fanout 广播模型

#### 1.编写生产者

```java
//注入rabbitTemplate
@Autowired
private RabbitTemplate rabbitTemplate;

//fanout 广播
@Test
public void testFanout() {
    rabbitTemplate.convertAndSend("logs", "", "Fanout的模型发送的消息");
}
```

#### 2.编写消费者-1

```java
@RabbitListener(bindings = {
    @QueueBinding(
        value = @Queue,//绑定临时队列
        exchange = @Exchange(value = "logs", type = "fanout") //绑定的交换机
    )
})
public void read1(String message) {
    System.out.println("message1="+message);
}
```

#### 3.编写消费者-2

```java
@RabbitListener(bindings = {
    @QueueBinding(
        value = @Queue,//绑定临时队列
        exchange = @Exchange(value = "logs", type = "fanout") //绑定的交换机
    )
})
public void read2(String message) {
    System.out.println("message2="+message);
}
```

### Routing 路由模型

#### 1.编写生产者

```java
//注入rabbitTemplate
@Autowired
private RabbitTemplate rabbitTemplate;

//routing 路由模式
@Test
public void testRoute() {
    rabbitTemplate.convertAndSend("directs", "info", "发送info的key的路由信息");
}
```

#### 2.编写消费者-1

```java
@RabbitListener(bindings = {
    @QueueBinding(
        value = @Queue, //创建临时队列
        exchange = @Exchange(value = "directs", type = "direct"), //自定义交换机名称和类型
        key = {"info", "error", "warn"}
    )
})
public void read1(String message) {
    System.out.println("message1==>" + message);
}
```

#### 3.编写消费者-2

```java
@RabbitListener(bindings = {
    @QueueBinding(
        value = @Queue, //创建临时队列
        exchange = @Exchange(value = "directs", type = "direct"), //自定义交换机名称和类型
        key = {"info"}
    )
})
public void read2(String message) {
    System.out.println("message1==>" + message);
}
```

### Topic 动态路由模型

#### 1.编写生产者

```java
//注入rabbitTemplate
@Autowired
private RabbitTemplate rabbitTemplate;

//topic 动态路由 订阅模式
@Test
public void testTopic() {
    rabbitTemplate.convertAndSend("topics", "user.save", "user.save 路由消息");
}
```

#### 2.编写消费者-1

```java
@RabbitListener(bindings = {
    @QueueBinding(
        value = @Queue,
        exchange = @Exchange(type = "topic", value = "topics"),
        key = {"user.save", "user.*"}
    )
})
public void read1(String message) {
    System.out.println("message1==>" + message);
}
```

#### 3.编写消费者-2

```java
@RabbitListener(bindings = {
    @QueueBinding(
        value = @Queue,
        exchange = @Exchange(type = "topic", value = "topics"),
        key = {"user.save", "user.*"}
    )
})
public void read2(String message) {
    System.out.println("message2==>" + message);
}
```

### MQ的应用场景

#### 1.异步处理

==场景说明：用户注册后，需要发注册邮件和注册短信，传统的做法有两种  1. 串行的方式  2. 并行的方式==

- **串行方式：**讲注册信息写入数据库后，发送注册邮件， 再发送注册短信，以上三个任务全部完成后才返回给客户端。这有一个问题是，邮件，短信并不是必须的，它只是一个通知，而这种做法让客户端等待没有必要等待没有必要等待的东西。
- **并行方式：**将信息写入数据库后，发送邮件的同时，发送短信，以上三个任务完成后，返回客户端，并行的方式能提高处理的时间。
- **消息队列：**假设三个业务点分别使用50ms，串行方式使用时间150ms，并行使用时间100ms。虽然并行已经提高了处理时间，但是，前面说过，邮件和短信不对我正常的使用网站没有任何影响，客户端没有必要等着其发送完成才显示注册成功，应该是写入数据库后就返回。引入消息队列后，把发送邮件，短信等不是必须的业务逻辑异步处理。

#### 2、应用解耦

==场景说明：双11是购物狂欢节，用户下单后，订单系统需要通知库存系统，传统的做法就是订单系统调用库存系统的接口==

这样做法有一个缺点：

当库存系统出现故障时，订单就会失效。订单系统和库存系统高耦合，引入消息队列

- **订单系统：**用户下单后，订单系统完成持久化处理，将消息写入消息队列，返回用户订单下单成功
- **库存系统：**订阅下单的消息，获取下单消息，进行库操作。就算库存系统出现故障，消息队列也能保证消息的可靠投递，不会导致消息丢失

#### 3、流量削锋

==场景说明：秒杀活动，一般会因为流量过大，导致应用挂掉，为了解决这个问题，一般在应用前端加入消息队列。==

作用：

1. 可以控制活动人数，超过此一定阈值的订单直接丢弃
2. 可以缓解短时间的高流量压垮应用
