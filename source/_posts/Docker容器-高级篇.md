---
title: Docker容器-高级篇
tags:
  - Docker
  - 容器
categories: Linux
slug: Docker-High
top: 1
cover: >-
  https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20211212170545.png
abbrlink: 289e2bff
date: 2022-04-07 22:02:35
updated: 2022-06-12 18:56:24
---

# Docker 容器-高级篇

## Docker 复杂应用安装

### MySQL 主从复制

主从搭建步骤：

- #### **新建主服务器容器实例3307**

```bash
docker run -p 3307:3306 --name mysql-master \
-v /mydata/mysql-master/log:/var/log/mysql \
-v /mydata/mysql-master/data:/var/lib/mysql \
-v /mydata/mysql-master/conf:/etc/mysql \
-e MYSQL_ROOT_PASSWORD=root  \
-d mysql:5.7
```

1. **进入/mydata/mysql-master/conf目录下新建my.cnf**

```bash
[root@88231 conf]# vim my.cnf
[mysqld]
## 设置server_id，同一局域网中需要唯一
server_id=101 
## 指定不需要同步的数据库名称
binlog-ignore-db=mysql  
## 开启二进制日志功能
log-bin=mall-mysql-bin  
## 设置二进制日志使用内存大小（事务）
binlog_cache_size=1M  
## 设置使用的二进制日志格式（mixed,statement,row）
binlog_format=mixed  
## 二进制日志过期清理时间。默认值为0，表示不自动清理。
expire_logs_days=7  
## 跳过主从复制中遇到的所有错误或指定类型的错误，避免slave端复制中断。
## 如：1062错误是指一些主键重复，1032错误是因为主从数据库数据不一致
slave_skip_errors=1062
```

2. **修改完配置后重启master实例**

```
[root@88231 conf]# docker restart mysql-master
mysql-master
```

3. **进入 mysql-master 容器**

```
[root@88231 conf]# docker exec -it mysql-master /bin/bash
root@c84fa378812d:/# mysql -uroot -p
Enter password:
```

4. **master 容器实例内创建数据同步用户**

```bash
mysql> CREATE USER 'slave'@'%' IDENTIFIED BY '123456';
Query OK, 0 rows affected (0.01 sec)

mysql> GRANT REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'slave'@'%';
Query OK, 0 rows affected (0.01 sec)
```

- #### 新建从服务器容器实例3308

```bash
docker run -p 3308:3306 --name mysql-slave \
-v /mydata/mysql-slave/log:/var/log/mysql \
-v /mydata/mysql-slave/data:/var/lib/mysql \
-v /mydata/mysql-slave/conf:/etc/mysql \
-e MYSQL_ROOT_PASSWORD=root  \
-d mysql:5.7
```

1. **进入/mydata/mysql-slave/conf目录下新建my.cnf**

```bash
[root@88231 conf]# vim my.cnf
[mysqld]
## 设置server_id，同一局域网中需要唯一
server_id=102
## 指定不需要同步的数据库名称
binlog-ignore-db=mysql  
## 开启二进制日志功能，以备Slave作为其它数据库实例的Master时使用
log-bin=mall-mysql-slave1-bin  
## 设置二进制日志使用内存大小（事务）
binlog_cache_size=1M  
## 设置使用的二进制日志格式（mixed,statement,row）
binlog_format=mixed  
## 二进制日志过期清理时间。默认值为0，表示不自动清理。
expire_logs_days=7  
## 跳过主从复制中遇到的所有错误或指定类型的错误，避免slave端复制中断。
## 如：1062错误是指一些主键重复，1032错误是因为主从数据库数据不一致
slave_skip_errors=1062  
## relay_log配置中继日志
relay_log=mall-mysql-relay-bin  
## log_slave_updates表示slave将复制事件写进自己的二进制日志
log_slave_updates=1  
## slave设置为只读（具有super权限的用户除外）
read_only=1
```

2. **修改完配置后重启slave实例**

```bash
[root@88231 conf]# docker restart mysql-slave
mysql-slave
```

3. **在主数据库中查看主从同步状态**

```bash
mysql> show master status;
+-----------------------+----------+--------------+------------------+-------------------+
| File                  | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+-----------------------+----------+--------------+------------------+-------------------+
| mall-mysql-bin.000001 |      617 |              | mysql            |                   |
+-----------------------+----------+--------------+------------------+-------------------+
1 row in set (0.01 sec)
```

4. **进入mysql-slave容器**

```bash
[root@88231 conf]# docker exec -it mysql-slave /bin/bash
root@820edd47f326:/# mysql -uroot -p
Enter password:
```

5. **在从数据库中配置主从复制**

```bash
change master to master_host='宿主机ip', master_user='slave', master_password='123456', master_port=3307, master_log_file='mall-mysql-bin.000001', master_log_pos=617, master_connect_retry=30;

mysql> change master to master_host='192.168.88.231', master_user='slave', master_password='123456', master_port=3307, master_log_file='mall-mysql-bin.000001', master_log_pos=617, master_connect_retry=30;
Query OK, 0 rows affected, 2 warnings (0.15 sec)

#主从复制命令参数说明
master_host：主数据库的IP地址；
master_port：主数据库的运行端口；
master_user：在主数据库创建的用于同步数据的用户账号；
master_password：在主数据库创建的用于同步数据的用户密码；
master_log_file：指定从数据库要复制数据的日志文件，通过查看主数据的状态，获取File参数；
master_log_pos：指定从数据库从哪个位置开始复制数据，通过查看主数据的状态，获取Position参数；
master_connect_retry：连接失败重试的时间间隔，单位为秒。
```

6. **在从数据库中查看主从同步状态**

```bash
mysql> show slave status \G;
*************************** 1. row ***************************
               Slave_IO_State: 
                  Master_Host: 192.168.88.231
                  Master_User: slave
                  Master_Port: 3307
                Connect_Retry: 30
              Master_Log_File: mall-mysql-bin.000001
          Read_Master_Log_Pos: 617
               Relay_Log_File: mall-mysql-relay-bin.000001
                Relay_Log_Pos: 4
        Relay_Master_Log_File: mall-mysql-bin.000001
         # NO -- 还没开始
             Slave_IO_Running: No
            Slave_SQL_Running: No
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 617
              Relay_Log_Space: 154
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: NULL
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 0
                  Master_UUID: 
             Master_Info_File: /var/lib/mysql/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: 
           Master_Retry_Count: 86400
                  Master_Bind: 
      Last_IO_Error_Timestamp: 
     Last_SQL_Error_Timestamp: 
               Master_SSL_Crl: 
           Master_SSL_Crlpath: 
           Retrieved_Gtid_Set: 
            Executed_Gtid_Set: 
                Auto_Position: 0
         Replicate_Rewrite_DB: 
                 Channel_Name: 
           Master_TLS_Version: 
1 row in set (0.00 sec)
```

7. **在从数据库中开启主从同步**

```bash
mysql> start slave;
Query OK, 0 rows affected (0.00 sec)
```

8. **查看从数据库状态发现已经同步**

```bash
mysql> show slave status \G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.88.231
                  Master_User: slave
                  Master_Port: 3307
                Connect_Retry: 30
              Master_Log_File: mall-mysql-bin.000001
          Read_Master_Log_Pos: 617
               Relay_Log_File: mall-mysql-relay-bin.000002
                Relay_Log_Pos: 325
        Relay_Master_Log_File: mall-mysql-bin.000001
           # Yes -- 已开始
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 617
              Relay_Log_Space: 537
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 101
                  Master_UUID: 25cb5d93-e4df-11ec-86da-0242ac110003
             Master_Info_File: /var/lib/mysql/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind: 
      Last_IO_Error_Timestamp: 
     Last_SQL_Error_Timestamp: 
               Master_SSL_Crl: 
           Master_SSL_Crlpath: 
           Retrieved_Gtid_Set: 
            Executed_Gtid_Set: 
                Auto_Position: 0
         Replicate_Rewrite_DB: 
                 Channel_Name: 
           Master_TLS_Version: 
1 row in set (0.00 sec)
```

9. **主从复制测试**

```
- 主机新建库-使用库-新建表-插入数据，ok
- 从机使用库-查看记录，ok
```

### Redis 集群

#### 搭建Redis集群

> 3主3从redis集群扩缩容配置案例架构说明
>
> https://www.processon.com/view/link/629e20255653bb03f2cc0a14

1. **新建6个docker容器redis实例**

```bash
docker run -d --name redis-node-1 --net host --privileged=true -v /data/redis/share/redis-node-1:/data redis:6.0.8 --cluster-enabled yes --appendonly yes --port 6381
 
docker run -d --name redis-node-2 --net host --privileged=true -v /data/redis/share/redis-node-2:/data redis:6.0.8 --cluster-enabled yes --appendonly yes --port 6382
 
docker run -d --name redis-node-3 --net host --privileged=true -v /data/redis/share/redis-node-3:/data redis:6.0.8 --cluster-enabled yes --appendonly yes --port 6383
 
docker run -d --name redis-node-4 --net host --privileged=true -v /data/redis/share/redis-node-4:/data redis:6.0.8 --cluster-enabled yes --appendonly yes --port 6384
 
docker run -d --name redis-node-5 --net host --privileged=true -v /data/redis/share/redis-node-5:/data redis:6.0.8 --cluster-enabled yes --appendonly yes --port 6385
 
docker run -d --name redis-node-6 --net host --privileged=true -v /data/redis/share/redis-node-6:/data redis:6.0.8 --cluster-enabled yes --appendonly yes --port 6386
```

==如果运行成功，效果如下：==

![image-20220606234533562](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220606234533.png)

> 命令分步解释：
>
> - docker run：创建并运行docker容器实例
>
> - --name redis-node-6：容器名字
>
> - --net host：使用宿主机的IP和端口，默认
>
> - --privileged=true：获取宿主机root用户权限
>
> - -v /data/redis/share/redis-node-6:/data：容器卷，宿主机地址:docker内部地址
>
> - redis:6.0.8：redis镜像和版本号
>
> - --cluster-enabled yes：开启redis集群
>
> - --appendonly yes：开启持久化
>
> - --port 6386：redis端口号

2. **进入容器 redis-node-1 并为 6 台机器构建集群关系**

```bash
# 进入容器
[root@88231 ~]# docker exec -it redis-node-1 /bin/bash

# 构建主从关系
# 注意，进入docker容器后才能执行一下命令，且注意自己的真实IP地址
redis-cli --cluster create 192.168.88.231:6381 192.168.88.231:6382 192.168.88.231:6383 192.168.88.231:6384 192.168.88.231:6385 192.168.88.231:6386 --cluster-replicas 1
# --cluster-replicas 1 表示为每个master创建一个slave节点
```

![image-20220606235150386](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220606235150.png)

![image-20220606235242475](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220606235242.png)

![image-20220606235332969](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220606235333.png)

==一切OK的话，3主3从搞定==

3. **链接进入6381作为切入点，查看集群状态**

```
root@88231:/data# redis-cli -p 6381
127.0.0.1:6381> keys *
127.0.0.1:6381> cluster info
127.0.0.1:6381> cluster nodes
```

<img src="https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220607000338.png" alt="image-20220607000338016" style="zoom:50%;" />	

![image-20220607000524306](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220607000524.png)

#### 主从容错切换迁移

##### 数据读写存储

1. 启动 6 个 redis 构成的集群并通过 exec 进入
2. 对 6381 新增两个 key
3. 防止路由失效加参数 -c 并新增两个 key

![image-20220609222228049](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220609222235.png)

![image-20220609222420357](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220609222420.png)

4. 查看集群信息

```bash
redis-cli --cluster check 192.168.88.231:6381
```

![image-20220609222613143](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220609222613.png)

##### 容错切换迁移

1. 主6381和从机切换，先停止主机6381
2. 6381主机停了，对应的真实从机上位
3. 6381作为1号主机分配的从机以实际情况为准，具体是几号机器就是几号
4. 再次查看集群信息

![image-20220609223136321](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220609223136.png)

> ==6381 宕机了，6385 上位成了新的 master==
>
> 备注：本次操作 6381 为主节点，对应的从节点是 6385，对应关系是随机的，每次操作以实际情况为准

5. 启动6381节点

```bash
docker start redis-node-1
```

![image-20220609224042839](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220609224043.png)

6. 再停6385节点

```bash
docker stop redis-node-5
```

![image-20220609224750744](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220609224750.png)

7. 再启6385节点

```bash
docker start redis-node-5
```

> ==发现主从节点又恢复之前的状态了==

8. 查看集群状态

```
redis-cli --cluster check 自己IP:6381

可以看到主节点分配的
```

![image-20220609230337692](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220609230337.png)

##### 主从扩容案例

1. 新建 6387、6388 两个节点+新建后启动+查看是否是 8 节点

```bash
docker run -d --name redis-node-7 --net host --privileged=true -v /data/redis/share/redis-node-7:/data redis:6.0.8 --cluster-enabled yes --appendonly yes --port 6387

docker run -d --name redis-node-8 --net host --privileged=true -v /data/redis/share/redis-node-8:/data redis:6.0.8 --cluster-enabled yes --appendonly yes --port 6388

docker ps
```

2. 进入 6387 容器实例内部

```bash
docker exec -it redis-node-7 /bin/bash
```

3. 将新增的 6387 节点（空槽号）作为 master 节点加入集群

```
redis-cli --cluster add-node 自己实际IP地址:6387 自己实际IP地址:6381
redis-cli --cluster add-node 192.168.88.231:6387 192.168.88.231:6381
6387 就是将要作为master新增节点
6381 就是原来集群节点里面的领路人，相当于6387拜拜6381的码头从而找到组织加入集群
```

![image-20220609231034192](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220609231034.png)

4. 检查集群情况第 1 次

```
redis-cli --cluster check 真实ip地址:6381

redis-cli --cluster check 192.168.88.231:6381
```

![image-20220609231423755](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220609231423.png)

5. 重新分派槽号

```
重新分派槽号

命令:redis-cli --cluster reshard IP地址:端口号

redis-cli --cluster reshard 192.168.88.231:6381
```

![image-20220609232741435](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220609232741.png)

6. 检查集群情况第 2 次

```bash
redis-cli --cluster check 真实ip地址:6381

redis-cli --cluster check 192.168.88.231:6381
```

![image-20220609233201196](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220609233201.png)

> ==槽号分派说明==
>
> 为什么6387是3个新的区间，以前的还是连续？
>
> 重新分配成本太高，所以前3家各自匀出来一部分，从6381/6382/6383三个旧节点分别匀出1364个坑位给新节点6387

![image-20220609233346240](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220609233346.png)

7. 为主节点 6387 分配从节点 6388

```
命令：redis-cli --cluster add-node ip:新slave端口 ip:新master端口 --cluster-slave --cluster-master-id 新主机节点ID
 
redis-cli --cluster add-node 192.168.88.231:6388 192.168.88.231:6387 --cluster-slave --cluster-master-id 7206137ce4e66c0464fa0fa00472202ce5b16792
-------这个是6387的编号，按照自己实际情况
```

![image-20220609233720722](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220609233720.png)

8. 检查集群第 3 次

```
redis-cli --cluster check 192.168.88.231:6382

4 主 4 从
```

![image-20220609233902550](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220609233902.png)

##### 主从缩容案例

> 目的：6387 和 6388 下线

1. 检查集群情况 - 获得 6388 的节点ID

```bash
redis-cli --cluster check 192.168.88.231:6382
```

![image-20220610223609176](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220610223609.png)

2. 将 6388 删除 从集群中将4号从节点 6388 删除

```bash
命令：redis-cli --cluster del-node ip:从机端口 从机6388节点ID
 
redis-cli --cluster del-node 192.168.88.231:6388 1fedf6a6f9acfbdba6951a532cd2d68e4546898e
```

![image-20220610223709085](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220610223709.png)

```bash
redis-cli --cluster check 192.168.88.231:6382
```

==检查一下发现，6388 被删除了，只剩下七台机器了。==

![image-20220610223900954](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220610223901.png)

3. 将 6387 的槽号清空，重新分配，本例将清出来的槽号都给 6381

```bash
redis-cli --cluster reshard 192.168.88.231:6381
```

![image-20220610224116496](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220610224116.png)

![image-20220610224657149](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220610224735.png)

> 将 6387 节点的槽号都分配给 6381

4. 检查集群情况

```bash
redis-cli --cluster check 192.168.88.231 6382

4096 个槽位都指给 6381，它变成了 8192 个槽位，相当于全部都给 6381了
```

![image-20220610225027291](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220610225027.png)

5. 删除 6387 节点

```bash
命令：redis-cli --cluster del-node ip:端口 6387节点ID
 
redis-cli --cluster del-node 192.168.88.231:6387 7206137ce4e66c0464fa0fa00472202ce5b16792
```

![image-20220610225144956](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220610225145.png)

6. 再次检查集群情况

```bash
redis-cli --cluster check 192.168.88.231 6382

恢复之前的 3 主 3 从，缩容成功！
```

![image-20220610225344237](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220610225344.png)

## DockerFile

### 1. Dockerfile 介绍

`Dockerfile` 是用来构建 Docker 镜像的文本文件，是由一条条构建镜像所需的指令和参数构成的脚本。

> 官网：https://docs.docker.com/engine/reference/builder/

构建步骤：

1. 编写一个 `Dockerfile` 文件
2. `docker bulid` 构建为一个镜像
3. `docker run` 运行镜像
4. `docker push` 发布镜像（DockerHub. 阿里云镜像仓库）

![image-20220611202748238](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220611202748.png)

### 2. Dockerfile 构建过程

#### 基础知识

1. 每个保留关键字（指令）都==必须是大写字母==且后面要跟随至少一个参数
2. 指令按照从上到下，顺序执行
3. `#` 表示注释
4. 每条指令都会创建一个新的镜像层并对镜像进行提交

![image-20211230224659329](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20211230224706.png)

> 总结
>
> 从应用软件的角度看，`Dockerfile`、`Docker镜像`与`Docker容器`分别代表软件的三个不同阶段：
>
> - `Dockerfile`是软件的原材料
> - `Docker镜像`是软件的交付品
> - `Docker容器`则可以认为是软件镜像的运行态，也即依照镜像运行的容器实例
>
> ==Dockerfile面向开发，Docker镜像成为交付标准，Docker容器则涉及部署与运维，三者缺一不可，合力充当Docker体系的基石==

![image-20220611204149366](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220611204149.png)

1. Dockerfile，需要定义一个Dockerfile，Dockerfile定义了进程需要的一切东西。Dockerfile涉及的内容包括执行代码或者是文件、环境变量、依赖包、运行时环境、动态链接库、操作系统的发行版、服务进程和内核进程(当应用进程需要和系统服务和内核进程打交道，这时需要考虑如何设计namespace的权限控制)等等;

2. Docker镜像，在用Dockerfile定义一个文件之后，docker build时会产生一个Docker镜像，当运行 Docker镜像时会真正开始提供服务;

3. Docker容器，容器是直接提供服务的。

### 3. DockerFile 的保留字指令

- `FROM`：基础镜像，当前新镜像是基于哪个镜像的，指定一个已经存在的镜像作为模板，第一条必须是FROM

- `MAINTAINER`：镜像维护者的姓名和邮箱地址

- `RUN`：容器构建时需要运行的命令，包含两种格式：

  - shell格式：

  ![image-20220611204952312](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220611204952.png)

  - exec格式：

  ![image-20220611205003786](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220611205003.png)

  - `RUN` 是在 `docker build` 时运行

- `EXPOSE`：当前容器对外暴露出的端口

- `WORKDIR`：指定在创建容器后，终端默认登陆的进来工作目录，一个落脚点

- `USER`：指定该镜像以什么样的用户去执行，如果都不指定，默认是root

- `ENV`：用来在构建镜像过程中设置环境变量

  - ```
    ENV MY_PATH /usr/mytest
    这个环境变量可以在后续的任何RUN指令中使用，这就如同在命令前面指定了环境变量前缀一样；
    也可以在其它指令中直接使用这些环境变量，
     
    比如：WORKDIR $MY_PATH
    ```

- `ADD`：将宿主机目录下的文件拷贝进镜像且会自动处理URL和解压tar压缩包

- `COPY`：类似ADD，拷贝文件和目录到镜像中

  - ```dockerfile
    # 将从构建上下文目录中 <源路径> 的文件/目录复制到新的一层的镜像内的 <目标路径> 位置
    
    COPY src dest
    COPY ["src", "dest"]
    
    # <src源路径>：源文件或者源目录
    # <dest目标路径>：容器内的指定路径，该路径不用事先建好，路径不存在的话，会自动创建。
    ```

- `VOLUME`：容器数据卷，用于数据保存和持久化工作

- `CMD`：指定容器启动后的要干的事情

  - ![image-20220611210408046](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220611210408.png)
  - ==注意：Dockerfile 中可以有多个 CMD 指令，但只有最后一个生效，CMD 会被 docker run 之后的参数替换==
  - 它和前面 `RUN` 命令的区别：
    - `CMD` 是在 `docker run` 时运行
    - `RUN` 是在 `docker build` 时运行

- `ENTRYPOINT`：也是用来指定一个容器启动时要运行的命令

  - 类似于 CMD 指令，==但是ENTRYPOINT不会被docker run后面的命令覆盖==， 而且这些命令行参数==会被当作参数送给 ENTRYPOINT 指令指定的程序==

### 4. 实战测试

`Docker Hub` 中 99% 的镜像都是从这个基础镜像构建而来的 -- FROM scratch

例如 `Centos`官方的 `DockerFile`

```shell
FROM scratch
ADD centos-7-x86_64-docker.tar.xz /

LABEL \
    org.label-schema.schema-version="1.0" \
    org.label-schema.name="CentOS Base Image" \
    org.label-schema.vendor="CentOS" \
    org.label-schema.license="GPLv2" \
    org.label-schema.build-date="20201113" \
    org.opencontainers.image.title="CentOS Base Image" \
    org.opencontainers.image.vendor="CentOS" \
    org.opencontainers.image.licenses="GPL-2.0-only" \
    org.opencontainers.image.created="2020-11-13 00:00:00+00:00"

CMD ["/bin/bash"]
```

#### 创建一个自己的 Centos 镜像

```shell
# 1. 编写Dockerfile文件
[root@ouwen dockerfile]# cat mydockerfile-centos
FROM centos
MAINTAINER luo_jj<2362766003@qq.com>

ENV MYPATH /user/local
WORKDIR $MYPATH

RUN yum -y install vim  # 添加vim工具
RUN yum -y install net-tools  #添加net-tools工具 ifconfig

EXPOSE 80

CMD echo $MYPATH
CMD echo "-----构建结束-----"
CMD /bin/bash

# 2. 通过这个文件来构建镜像
[root@ouwen dockerfile]# docker build -f dockerfile文件路径 -t 镜像名:[tag] .
Successfully built 71affc0619cf
Successfully tagged mycentos:1.0

# 3. 测试运行
[root@ouwen dockerfile]# docker run -it mycentos:1.0

# 4. 查看镜像的构建过程
[root@ouwen dockerfile]# docker history 71affc0619cf
IMAGE          CREATED         CREATED BY                                      SIZE      COMMENT
71affc0619cf   4 minutes ago   /bin/sh -c #(nop)  CMD ["/bin/sh" "-c" "/bin…   0B
e57701116948   4 minutes ago   /bin/sh -c #(nop)  CMD ["/bin/sh" "-c" "echo…   0B
5320ae8e4e26   4 minutes ago   /bin/sh -c #(nop)  CMD ["/bin/sh" "-c" "echo…   0B
99dab79f91c0   4 minutes ago   /bin/sh -c #(nop)  EXPOSE 80                    0B
5e112e2ce789   4 minutes ago   /bin/sh -c yum -y install net-tools             27.3MB
78e4cb3e3f0a   4 minutes ago   /bin/sh -c yum -y install vim                   64.8MB
13491c8aaa57   4 minutes ago   /bin/sh -c #(nop) WORKDIR /user/local           0B
aa37dd2d70e0   4 minutes ago   /bin/sh -c #(nop)  ENV MYPATH=/user/local       0B
5387147d0546   4 minutes ago   /bin/sh -c #(nop)  MAINTAINER luo_jj<2362766…   0B
5d0da3dc9764   3 months ago    /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B
<missing>      3 months ago    /bin/sh -c #(nop)  LABEL org.label-schema.sc…   0B
<missing>      3 months ago    /bin/sh -c #(nop) ADD file:805cb5e15fb6e0bb0…   231MB
```

#### 对比 CMD 和 ENTRYPOINT

```shell
# 1. 创建一个dockerfile文件
[root@ouwen dockerfile]# vim dockerfile-cmd-test
[root@ouwen dockerfile]# cat dockerfile-cmd-test
FROM centos
CMD ["ls","-a"] # 启动时执行 ls -a
# 2. 构建镜像
[root@ouwen dockerfile]# docker build -f dockerfile-cmd-test -t cmdtest:1.0 .
Sending build context to Docker daemon  3.072kB
Step 1/2 : FROM centos
 ---> 5d0da3dc9764
Step 2/2 : CMD ["ls","-a"]
 ---> Running in dcc500c7090b
Removing intermediate container dcc500c7090b
 ---> 6bcf9259bdb3
Successfully built 6bcf9259bdb3
Successfully tagged cmdtest:1.0
# 3. 启动测试 ls -a 生效了
[root@ouwen dockerfile]# docker run cmdtest:1.0
.
..
.dockerenv
bin
dev
etc
home
lib
lib64
lost+found
media
mnt
opt
proc
root
run
sbin
srv
sys
tmp
usr
var
# 继续测试，启动时追加 -l 命令 = ls -al
[root@ouwen dockerfile]# docker run cmdtest:1.0 -l
docker: Error response from daemon: OCI runtime create failed: container_linux.go:380: starting container process caused: exec: "-l": executable file not found in $PATH: unknown.
ERRO[0000] error waiting for container: context canceled
# 发现报错 原因是 使用 CMD ["ls","-a"] 的情况下，追加 -l 会替换 CMD ["ls","-a"] ，而 -l 不是能独立运行的命令
# 正确的做法：
[root@ouwen dockerfile]# docker run cmdtest:1.0 ls -al
total 56
drwxr-xr-x   1 root root 4096 Dec 30 15:45 .
drwxr-xr-x   1 root root 4096 Dec 30 15:45 ..
-rwxr-xr-x   1 root root    0 Dec 30 15:45 .dockerenv
lrwxrwxrwx   1 root root    7 Nov  3  2020 bin -> usr/bin
drwxr-xr-x   5 root root  340 Dec 30 15:45 dev
drwxr-xr-x   1 root root 4096 Dec 30 15:45 etc
drwxr-xr-x   2 root root 4096 Nov  3  2020 home
lrwxrwxrwx   1 root root    7 Nov  3  2020 lib -> usr/lib
lrwxrwxrwx   1 root root    9 Nov  3  2020 lib64 -> usr/lib64
drwx------   2 root root 4096 Sep 15 14:17 lost+found
drwxr-xr-x   2 root root 4096 Nov  3  2020 media
drwxr-xr-x   2 root root 4096 Nov  3  2020 mnt
drwxr-xr-x   2 root root 4096 Nov  3  2020 opt
dr-xr-xr-x 101 root root    0 Dec 30 15:45 proc
dr-xr-x---   2 root root 4096 Sep 15 14:17 root
drwxr-xr-x  11 root root 4096 Sep 15 14:17 run
lrwxrwxrwx   1 root root    8 Nov  3  2020 sbin -> usr/sbin
drwxr-xr-x   2 root root 4096 Nov  3  2020 srv
dr-xr-xr-x  13 root root    0 Dec 30 15:45 sys
drwxrwxrwt   7 root root 4096 Sep 15 14:17 tmp
drwxr-xr-x  12 root root 4096 Sep 15 14:17 usr
drwxr-xr-x  20 root root 4096 Sep 15 14:17 var

```

测试 ENTRYPOINT

```shell
# 1. 创建dockerfile文件
[root@ouwen dockerfile]# vim dockerfile-entrypoint-test
[root@ouwen dockerfile]# cat dockerfile-entrypoint-test
FROM centos
ENTRYPOINT ["ls","-a"] # 启动时执行 ls -a

# 2. 构建镜像
[root@ouwen dockerfile]# docker build -f dockerfile-entrypoint-test -t entrypointtest:1.0 .
Sending build context to Docker daemon  4.096kB
Step 1/2 : FROM centos
 ---> 5d0da3dc9764
Step 2/2 : ENTRYPOINT ["ls","-a"]
 ---> Running in cc256ca1c456
Removing intermediate container cc256ca1c456
 ---> 30cb5f7d0494
Successfully built 30cb5f7d0494
Successfully tagged entrypointtest:1.0

# 3. 启动测试
[root@ouwen dockerfile]# docker run entrypointtest:1.0
.
..
.dockerenv
bin
dev
etc
home
lib
lib64
lost+found
media
mnt
opt
proc
root
run
sbin
srv
sys
tmp
usr
var

# 4. 追加 -l 命令 没有问题！
[root@ouwen dockerfile]# docker run entrypointtest:1.0 -l
total 56
drwxr-xr-x   1 root root 4096 Dec 30 15:47 .
drwxr-xr-x   1 root root 4096 Dec 30 15:47 ..
-rwxr-xr-x   1 root root    0 Dec 30 15:47 .dockerenv
lrwxrwxrwx   1 root root    7 Nov  3  2020 bin -> usr/bin
drwxr-xr-x   5 root root  340 Dec 30 15:47 dev
drwxr-xr-x   1 root root 4096 Dec 30 15:47 etc
drwxr-xr-x   2 root root 4096 Nov  3  2020 home
lrwxrwxrwx   1 root root    7 Nov  3  2020 lib -> usr/lib
lrwxrwxrwx   1 root root    9 Nov  3  2020 lib64 -> usr/lib64
drwx------   2 root root 4096 Sep 15 14:17 lost+found
drwxr-xr-x   2 root root 4096 Nov  3  2020 media
drwxr-xr-x   2 root root 4096 Nov  3  2020 mnt
drwxr-xr-x   2 root root 4096 Nov  3  2020 opt
dr-xr-xr-x 100 root root    0 Dec 30 15:47 proc
dr-xr-x---   2 root root 4096 Sep 15 14:17 root
drwxr-xr-x  11 root root 4096 Sep 15 14:17 run
lrwxrwxrwx   1 root root    8 Nov  3  2020 sbin -> usr/sbin
drwxr-xr-x   2 root root 4096 Nov  3  2020 srv
dr-xr-xr-x  13 root root    0 Dec 30 15:45 sys
drwxrwxrwt   7 root root 4096 Sep 15 14:17 tmp
drwxr-xr-x  12 root root 4096 Sep 15 14:17 usr
drwxr-xr-x  20 root root 4096 Sep 15 14:17 var

```

#### 实战：Tomcat 镜像

1. 准备镜像文件：tomcat 压缩包. jdk 压缩包

   ![image-20220101235205728](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220101235212.png)

2. 编写 dockerfile 文件，官方命名 `Dockerfile`，`build` 时会自动寻找这个文件，不需要 `-f` 指定！

   ```shell
   [root@ouwen ouwen]# vim Dockerfile
   FROM centos
   MAINTAINER luo_jj<2362766003@qq.com>

   COPY readme.txt /usr/local/readme.txt

   ADD jdk-8u161-linux-x64.tar.gz /usr/local
   ADD apache-tomcat-8.5.55.tar.gz /usr/local

   RUN yum -y install vim

   ENV MYPATH /usr/local
   WORKDIR $MYPATH

   ENV JAVA_HOME /usr/local/jdk1.8.0_161
   ENV CLASSPATH $JAVA_HOME/lib/dt.jar;$JAVA_HOME/lib/tools.jar
   ENV CATALINA_HOME /usr/local/apache-tomcat-8.5.55
   ENV CATALINA_BASE /usr/local/apache-tomcat-8.5.55
   ENV PATH $PATH;$JAVA_HOME;$CLASSPATH;$CATALINA_HOME/lib;$CATALINA_BASE/bin

   EXPOSE 8080

   CMD /usr/local/apache-tomcat-8.5.55/bin/startup.sh && tail -F /usr/local/apache-tomcat-8.5.55/bin/logs/catalina.out
   ```

3. 构建镜像

   ```shell
   [root@ouwen ouwen]# docker build -t diytomcat .
   ```

4. 启动镜像

   ```shell
   [root@ouwen ouwen]# docker run -d -p 8080:8080 --name ouwentomcat -v /home/ouwen/tomcat/test:/usr/local/apache-tomcat-8.5.55/webapps/test -v /home/ouwen/tomcat/logs/:/usr/local/apache-tomcat-8.5.55/logs diytomcat
   163ad2d5705255a5aa2fd022467563a3324980b66beac56be04720cf45d0acba
   ```

5. 访问测试

   ![image-20220102005215449](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220102005215.png)

6. 发布项目（由于做了卷挂载，我们可以直接在本地目录编写项目就可以发布了！）

   手动编写一个 web 项目测试！

   在主机挂载的项目目录：

   ```shell
   # 1. 创建 WEB-INF 文件夹
   [root@ouwen test]# mkdir WEB-INF
   # 2. 编写 web.xml 文件
   [root@ouwen WEB-INF]# vim web.xml
   # 3. 编写首页 index.jsp
   [root@ouwen test]# vim index.jsp
   ```

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <web-app version="2.4"
       xmlns="http://java.sun.com/xml/ns/j2ee"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee
           http://java.sun.com/xml/ns/j2ee/web-app_2_4.xsd">
   </web-app>
   ```

   ```html
   <%@ page language="java" contentType="text/html; charset=UTF-8"
       pageEncoding="UTF-8"%>
   <!DOCTYPE html>
   <html>
   <head>
   <meta charset="utf-8">
   <title>菜鸟教程(runoob.com)</title>
   </head>
   <body>
   Hello World!<br/>
   <%
   out.println("你的 IP 地址 " + request.getRemoteAddr());
   %>
   </body>
   </html>
   ```

7. 访问测试，成功！

   ![image-20220102011951322](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220102011951.png)

### 5. 发布自己的镜像

> DockerHub

1. 地址 https://hub.docker.com/ 注册自己的账号！

2. 确保这个账号可以登录

3. 在服务器上提交自己的镜像

   ```shell
   [root@ouwen ~]# docker login --help

   Usage:  docker login [OPTIONS] [SERVER]

   Log in to a Docker registry.
   If no server is specified, the default is defined by the daemon.

   Options:
     -p, --password string   Password
         --password-stdin    Take the password from stdin
     -u, --username string   Username
   [root@ouwen ~]# docker login -u ouwen666
   Password:
   WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
   Configure a credential helper to remove this warning. See
   https://docs.docker.com/engine/reference/commandline/login/#credentials-store

   Login Succeeded
   ```

4. 使用 `docker login` 登录之后就可以提交镜像了

   ```shell
   # 1. 使用 docker tag 命令修改镜像版本
   [root@ouwen ~]# docker tag 352abc3918b1 ouwen666/tomcat:1.0
   # 2. 使用 docker push 命令提交镜像到 DockerHub
   [root@ouwen ~]# docker push ouwen666/tomcat:1.0
   ```

   ![image-20220103135111272](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220103135111.png)

   > 发现：提交的时候也是按照镜像的层级来的！

> 阿里云镜像

1. 登录阿里云

2. 找到容器镜像服务

3. 创建镜像仓库

   ![image-20220103140835560](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220103140835.png)

4. 浏览仓库信息

   ![image-20220103140926596](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220103140926.png)

### 6. 总结

![image-20220103142038290](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220103142038.png)

## Docker 网络

### 1. 理解`docker0`网卡

使用 `ip addr` 查看本机 ip

![image-20220103235821684](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220104001129.png)

> 发现三个网卡

**问题：docker 是如何处理容器网络访问的？**

```bash
# 启动一个tomcat容器
[root@ouwen ~]# docker run -d -P --name tomcat01 tomcat

# 查看容器的内部网卡信息
[root@ouwen ~]# docker exec -it tomcat01 ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
6: eth0@if7: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever

# 若没有 ip addr 命令可以执行 apt install 安装 iproute2
apt update && apt install -y iproute2

# linux主机可以ping通容器内部！
[root@ouwen ~]# ping 172.17.0.2
PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.037 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.040 ms
```

**发现：**

- 我们每启动一个 docker 容器，docker 就会给容器分配一个 ip，我们只要安装了 docker，就会有一个网卡 docker0 桥接模式，使用的技术是 `veth-pari` 技术！

![image-20220327220908091](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220327220915.png)

再启动一个容器，查看 `ip addr`，发现又多了一对网卡！

![image-20220327221202686](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220327221202.png)

> 我们发现这个容器生成的网卡都是成对出现的！
>
> veth-pari 就是一对虚拟设备接口，他们都是成对出现的，一端连着协议，一端彼此相连
>
> [Linux 虚拟网络设备 veth-pair 详解](https://www.cnblogs.com/bakari/p/10613710.html)

- docker 容器与容器之间是可以 ping 通的（同一网段下）

<mark>所有的容器在不指定网络的情况下，都是 docker0 路由的，docker 会给我们的容器分配一个可用的 IP</mark>

> 总结：docker 网络采用的是 Linux 桥接模式，通过 veth 成对建立连接

### 2. --link

在项目中，当数据库与应用不是在一台服务器上时，配置`database url=ip:xxx`，若数据库 ip 更换了，需要手动更改配置，非常的不友好，那么如何解决这个问题？

```
## 尝试使用容器名来ping，发现无法ping通
[root@ouwen ~]# docker exec -it tomcat02 ping tomcat01
ping: tomcat01: Name or service not known

## 若没有ping命令，执行以下命令安装：
apt-get update && apt install iputils-ping

## 启动tomcat03，通过--link连接tomcat02
[root@ouwen ~]# docker run -d -P --name tomcat03 --link tomcat02 tomcat

## 使用tomcat03通过ping容器名，成功ping通
[root@ouwen ~]# docker exec -it tomcat03 ping tomcat02
PING tomcat02 (172.17.0.3) 56(84) bytes of data.
64 bytes from tomcat02 (172.17.0.3): icmp_seq=1 ttl=64 time=0.066 ms
64 bytes from tomcat02 (172.17.0.3): icmp_seq=2 ttl=64 time=0.040 ms
64 bytes from tomcat02 (172.17.0.3): icmp_seq=3 ttl=64 time=0.041 ms

## 通过docker inspect命令查看tomcat03容器详情
[root@ouwen ~]# docker inspect tomcat03
## 发现Links配置如下图
```

<img src="https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220402223608.png" alt="image-20220402223601432" style="zoom: 33%;" />

**原理分析**

```
## 进入tomcat03容器，查看容器内的hosts文件
[root@ouwen ~]# docker exec -it tomcat03 /bin/bash
root@db6ba115385c:/usr/local/tomcat# cat /etc/hosts
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
ff00::0	ip6-mcastprefix
ff02::1	ip6-allnodes
ff02::2	ip6-allrouters
172.17.0.3	tomcat02 29285f35bb36
172.17.0.4	db6ba115385c
```

> --link 配置其实就是在容器内部配置了 hosts 文件，将容器名称绑定到指定 IP
>
> 这种方式已经是过时！不建议使用！

### 3. 自定义网络

**帮助命令**

```
docker network --help
```

**查看所有的 docker 网络**

```
[root@ouwen ~]# docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
f94736cb6a60   bridge    bridge    local
86b125d30748   host      host      local
a34af85c010b   none      null      local
```

**常见的网络模式**

| Docker 网络模式 | 配置                      | 说明                                                                                                    |
| --------------- | ------------------------- | ------------------------------------------------------------------------------------------------------- |
| host 模式       | –net=host                 | 容器和宿主机共享 Network namespace。                                                                    |
| container 模式  | –net=container:NAME_or_ID | 容器和另外一个容器共享 Network namespace。 kubernetes 中的 pod 就是多个容器共享一个 Network namespace。 |
| none 模式       | –net=none                 | 容器有独立的 Network namespace，但并没有对其进行任何网络设置，如分配 veth pair 和网桥连接，配置 IP 等。 |
| bridge 模式     | –net=bridge               | （默认为该模式）                                                                                        |

**创建一个自定义网络**

```
## 创建一个自定义网络mynet
# --driver bridge 桥接模式
# --subnet 192.168.0.0/16 子网掩码
# --gateway 192.168.0.1 网关地址
[root@ouwen ~]# docker network create --driver bridge --subnet 192.168.0.0/16 --gateway 192.168.0.1 mynet
7d74345ad8e8b909f012acb656cb3ee3e16425f5c952b18467efcea5b939d4ee
[root@ouwen ~]# docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
f94736cb6a60   bridge    bridge    local
86b125d30748   host      host      local
7d74345ad8e8   mynet     bridge    local
a34af85c010b   none      null      local
```

查看网络详情，创建完毕

<img src="https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220402225446.png" alt="image-20220402225446743" style="zoom:33%;" />

通过自定义网络启动容器

```
## 启动两个tomcat容器
[root@ouwen ~]# docker run -d -P --name tomcat-net-01 --net mynet tomcat
[root@ouwen ~]# docker run -d -P --name tomcat-net-02 --net mynet tomcat

```

![image-20220402225814526](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220402225814.png)

> 自定义网络可以实现--link 的功能，可以 ping 容器名！
>
> 好处：搭建多个集群时，可以创建多个网络，互相隔离开！

### 4. 网络连通

**帮助命令：**

```
docker network connect --help
```

将默认 docker0 网络下的容器添加到自定义网络 mynet 中，实现网络连通

```
[root@ouwen ~]# docker network connect mynet tomcat01
```

查看自定义网络详情：

```
[root@ouwen ~]# docker network inspect mynet
```

![image-20220402231507159](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220402231507.png)

> 发现将 docker0 中的 tomcat01 添加到 mynet 网络中来，并分配了一个 ip


## Docker Compose

> 官方文档：https://docs.docker.com/compose/

### 什么是 Docker Compose

`Docker Compose` 是一个用于定义和运行多容器 `Docker` 应用程序的工具。使用 `Compose`，您可以使用 `YAML` 文件来配置应用程序的服务。然后，使用单个命令，从配置创建并启动所有服务

使用 `Docker Compose` 基本上有以下三步：

1. 使用 定义应用的环境，以便可以在任何位置重现它。`Dockerfile`
2. 定义构成应用的服务，以便它们可以在隔离的环境中一起运行。`docker-compose.yml`
3. 运行[Docker Compose](https://docs.docker.com/compose/cli-command/)将启动并运行整个应用。您也可以使用 docker-compose 二进制文件运行。` docker compose up``docker-compose up `

`docker-compose.yml` 示例：

```yaml
version: "3.9"  # optional since v1.27.0
services:
  web:
    build: .
    ports:
      - "8000:5000"
    volumes:
      - .:/code
      - logvolume01:/var/log
    links:
      - redis
  redis:
    image: redis
volumes:
  logvolume01: {}
```

### 安装 Docker Compose

1. 下载

```bash
 # 官网地址
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

 # 国内镜像
sudo curl -L "https://get.daocloud.io/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

```

![image-20220404124103270](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220404124103.png)

2. 给`docker-compose`文件授可执行权限

```bash
sudo chmod +x /usr/local/bin/docker-compose
```

3. 测试安装是否成功

```
docker-compose --version
```

![image-20220404124418121](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220404124418.png)

### Docker Compose 入门

官方文档：https://docs.docker.com/compose/gettingstarted/

参考官方文档，运行一个简单的 `Python Web` 应用程序。该应用程序使用 `Flask` 框架，并在 `Redis` 中维护一个命中计数器

**步骤：**

**准备工作**

1. 创建一个项目文件夹

```
mkdir composetest
cd composetest
```

2. 创建一个在项目目录中调用的文件，并将其粘贴到：`app.py`

```bash
vim app.py
```

`app.py` 文件内容：

```python
import time

import redis
from flask import Flask

app = Flask(__name__)
cache = redis.Redis(host='redis', port=6379)

def get_hit_count():
    retries = 5
    while True:
        try:
            return cache.incr('hits')
        except redis.exceptions.ConnectionError as exc:
            if retries == 0:
                raise exc
            retries -= 1
            time.sleep(0.5)

@app.route('/')
def hello():
    count = get_hit_count()
    return 'Hello World! I have been seen {} times.\n'.format(count)
```

3. 创建在项目目录中调用的另一个文件，并将其粘贴到：`requirements.txt`

```bash
vim requirements.txt
```

`requirements.txt` 文件内容：

```
flask
redis
```

**创建 Dockerfile**

在项目目录中创建一个 `Dockerfile` 文件，并粘贴以下内容

```
FROM python:3.7-alpine
WORKDIR /code
ENV FLASK_APP=app.py
ENV FLASK_RUN_HOST=0.0.0.0
RUN apk add --no-cache gcc musl-dev linux-headers
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt
EXPOSE 5000
COPY . .
CMD ["flask", "run"]
```

说明：

- 从 `Python 3.7` 映像开始构建映像。

- 将工作目录设置为 `/code`
- 设置命令使用的环境变量`flask`
- 安装 `gcc` 和其他依赖项
- 复制并安装 Python 依赖项 `requirements.txt`
- 向映像添加元数据以描述容器正在侦听端口 5000
- 将项目中的当前目录复制到映像中的 `workdir`
- 将容器的缺省命令设置为 `flask run`

**编辑一个** `docker-compose.yml` **文件，编写服务**

创建在项目目录中调用的文件并粘贴以下内容：`docker-compose.yml`

```yml
version: "3.9"
services:
  web:
    build: .
    ports:
      - "8000:5000"
  redis:
    image: "redis:alpine"
```

这个 `docker-compose.yml` 定义了两个服务：`web` 和 `redis`

**使用 Compose 构建并运行项目**

在项目目录中，通过运行 `docker-compose up` 命令启动项目

```
docker-compose up
```

![image-20220404184408949](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220404184409.png)

![image-20220404195214755](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220404195214.png)

**启动成功，访问测试：**

![image-20220404195828215](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220404195828.png)

### 常用命令

- **ps**：列出所有运行容器

```bash
docker-compose ps
```

- **logs**：查看服务日志输出

```bash
docker-compose logs
```

- **port**：打印绑定的公共端口，下面命令可以输出 eureka 服务 8761 端口所绑定的公共端口

```bash
docker-compose port eureka 8761
```

- **build**：构建或者重新构建服务

```bash
docker-compose build
```

- **start**：启动指定服务已存在的容器

```bash
docker-compose start eureka
```

- **stop**：停止已运行的服务的容器

```bash
docker-compose stop eureka
```

- **rm**：删除指定服务的容器

```bash
docker-compose rm eureka
```

- **up**：构建、启动容器

```bash
docker-compose up
```

- **kill**：通过发送 SIGKILL 信号来停止指定服务的容器

```bash
docker-compose kill eureka
```

- **pull**：下载服务镜像
- **scale**：设置指定服务运气容器的个数，以 service=num 形式指定

```bash
docker-compose scale user=3 movie=3
```

- **run**：在一个服务上执行一个命令

```bash
docker-compose run web bash
```

### docker-compose.yml 文件规则

官网地址：https://docs.docker.com/compose/compose-file/compose-file-v3/

- **version**：指定 docker-compose.yml 文件的写法格式
- **services**：服务，多个容器集合
- **build**：配置构建时，Compose 会利用它自动构建镜像，该值可以是一个路径，也可以是一个对象，用于指定 Dockerfile 参数

```yaml
build: ./dir
---------------
build:
    context: ./dir
    dockerfile: Dockerfile
    args:
        buildno: 1
```

- **command**：覆盖容器启动后默认执行的命令

```yaml
command: bundle exec thin -p 3000
----------------------------------
command: [bundle,exec,thin,-p,3000]
```

- **dns**：配置 dns 服务器，可以是一个值或列表

```yaml
dns: 8.8.8.8
------------
dns:
    - 8.8.8.8
    - 9.9.9.9
```

- **dns_search**：配置 DNS 搜索域，可以是一个值或列表

```yaml
dns_search: example.com
------------------------
dns_search:
    - dc1.example.com
    - dc2.example.com
```

- **environment**：环境变量配置，可以用数组或字典两种方式

```yaml
environment:
    RACK_ENV: development
    SHOW: 'ture'
-------------------------
environment:
    - RACK_ENV=development
    - SHOW=ture
```

- **env_file**：从文件中获取环境变量，可以指定一个文件路径或路径列表，其优先级低于 environment 指定的环境变量

```yaml
env_file: .env
---------------
env_file:
    - ./common.env
```

- **expose**：暴露端口，只将端口暴露给连接的服务，而不暴露给主机

```yaml
expose:
    - "3000"
    - "8000"
```

- **image**：指定服务所使用的镜像

```yaml
image: java
```

- **network_mode**：设置网络模式

```yaml
network_mode: "bridge"
network_mode: "host"
network_mode: "none"
network_mode: "service:[service name]"
network_mode: "container:[container name/id]"
```

- **ports**：对外暴露的端口定义，和 expose 对应

```yaml
ports:   # 暴露端口信息  - "宿主机端口:容器暴露端口"
- "8763:8763"
- "8763:8763"
```

- **links**：将指定容器连接到当前连接，可以设置别名，避免 ip 方式导致的容器重启动态改变的无法连接情况

```yaml
links:    # 指定服务名称:别名
    - docker-compose-eureka-server:compose-eureka
```

- **volumes**：卷挂载路径

```yaml
volumes:
  - /lib
  - /var
```

- **logs**：日志输出信息

```yaml
--no-color          单色输出，不显示其他颜.
-f, --follow        跟踪日志输出，就是可以实时查看日志
-t, --timestamps    显示时间戳
--tail              从日志的结尾显示，--tail=200
```

## Docker Swarm

> 官方文档：https://docs.docker.com/engine/swarm/

### 什么是 Docker Swarm

Swarm 是 Docker 公司推出的用来管理 docker 集群的平台，它是将一群 Docker 宿主机变成一个单一的虚拟主机，这大大方便了用户将原本基于单节点的系统移植到 Swarm 上，同时 Swarm 内置了对 Docker 网络插件的支持，用户也很容易的部署跨主机的容器集群服务。

Docker Swarm 和 Docker Compose 一样，都是 Docker 官方容器编排项目，但不同的是，Docker Compose 是一个在单个服务器或主机上创建多个容器的工具，而 Docker Swarm 则可以在多个服务器或主机上创建容器集群服务。

### Docker Swarm 架构图

![image-20220405181654649](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220405181654.png)

上图可以看出，Swarm 是典型的 master-slave 结构，通过发现服务来选举 manager。manager 是中心管理节点，各个 node 上运行 agent 接受 manager 的统一管理，集群会自动通过 Raft 协议分布式选举出 manager 节点，无需额外的发现服务支持，避免了单点的瓶颈问题，同时也内置了 DNS 的负载均衡和对外部负载均衡机制的集成支持。

### Swarm 的几个关键概念

- Swarm

`Swarm` 由多个 `Docker` 主机组成，这些主机以 `Swarm模式` 运行，并充当 `managers`（用于管理集群中的节点）和 `workers`（运行 Swarm 服务）。一个 `Docker 主机` 可以是`manager`、`work`，也可以同时成为这两个角色。创建服务时，需要定义其属性（副本数、可供其使用的网络和存储资源、服务向外界公开的端口等）。Docker 致力于维护所需的属性。例如，如果工作线程节点变得不可用，Docker 会将该节点的`Task`安排在其他节点上。`Task`是一个正在运行的容器，它是 `Swarm` 服务的一部分，由`Swarm manager`管理，而不是独立的容器。

- Node

`Node`是参与群的 Docker 引擎的一个实例。您也可以将其视为 Docker 节点。您可以在一台物理计算机或云服务器上运行一个或多个节点，但生产群部署通常包括分布在多台物理和云计算机上的 Docker 节点。

要将应用程序部署到 swarm，请将服务定义提交给 管理器节点。管理器节点将称为任务的工作单元分派给工作节点。

管理器节点还执行维护群所需状态所需的编排和集群管理功能。管理器节点选择单个领导者来执行编排任务。

工作线程节点接收并执行从管理器节点分派的任务。缺省情况下，管理器节点也作为工作线程节点运行服务，但您可以将它们配置为以独占方式运行管理器任务，并且是仅限管理器的节点。代理在每个工作节点上运行，并报告分配给它的任务。工作线程节点通知管理器节点其分配的任务的当前状态，以便管理器可以维护每个工作线程的所需状态。

- Service

一个服务是任务的定义，管理机或工作节点上执行。它是群体系统的中心结构，是用户与群体交互的主要根源。

创建服务时，你需要指定要使用的容器镜像。

- Task

任务是在 docekr 容器中执行的命令，Manager 节点根据指定数量的任务副本分配任务给 worker 节点

### Swarm 的工作模式

Swarm 集群由管理节点（manager）和工作节点（work node）构成。

- swarm manage：负责整个集群的管理工作包括集群配置、服务管理等所有跟集群有关的工作
- work node：即图中的 avaliable node，主要负责运行相应的服务来执行任务（task）

![image-20220405204714347](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220405204738.png)

### 常用命令

#### 创建 Swarm

- ##### 创建第一个 Node

```bash
docker swarm init --advertise-addr $IP
```

`$IP` 是当前 Node 的外部可访问 IP，便于其他 Node 寻址。这样，一个 Swarm 就被初始化完成了，它仅有一个 Manager 节点。

- ##### 查看 Swarm 状态

```bash
docker info
```

- ##### 查看节点信息

```bash
docker node ls
```

#### 将节点添加到 Swarm 中

- ##### 添加新的 Node 到 Swarm

在 Manager 节点，执行以下命令可查看到如何放入一个 Node：

```bash
$ join-token manager
To add a manager to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-2ngicgfdko2f95o108z5f32lcop5eertv2ykvhv6fhrpia9zri-bikzx84znfiatib0279jusn5i 192.168.88.86:2377

$ docker swarm join-token worker
To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-2ngicgfdko2f95o108z5f32lcop5eertv2ykvhv6fhrpia9zri-4a7t10ktyl1u055gqyoxjh6uj 192.168.88.86:2377
```

在一个未加入任何`Swarm`的机器上，执行以上显示的命令 `docker swarm join --token...`，即可称为这个 `Swarm` 的 `Manager`或`Worker`节点

#### 部署服务

- ##### 创建服务

进入运行管理器节点的计算机，运行以下命令：

```bash
$ docker service create --replicas 1 --name helloworld alpine ping docker.com
ktcjndh4nj4nbwp461idnob0y
```

1. 该命令将创建服务。`docker service create`
2. 该参数命名服务 。` --name``helloworld `
3. 该参数指定 1 个正在运行的实例的所需状态。`--replicas`
4. 这些参数将服务定义为执行命令的 Alpine Linux 容器。`alpine ping docker.com`

- ##### 查看正在运行的服务

```bash
$ docker service ls
ID             NAME         MODE         REPLICAS   IMAGE           PORTS
ktcjndh4nj4n   helloworld   replicated   1/1        alpine:latest
```

#### 查看服务

- ##### 查看服务详细信息

```bash
$ docker service inspect --pretty helloworld

ID:		ktcjndh4nj4nbwp461idnob0y
Name:		helloworld
Service Mode:	Replicated
 Replicas:	1
Placement:
UpdateConfig:
 Parallelism:	1
 On failure:	pause
 Monitoring Period: 5s
 Max failure ratio: 0
 Update order:      stop-first
RollbackConfig:
 Parallelism:	1
 On failure:	pause
 Monitoring Period: 5s
 Max failure ratio: 0
 Rollback order:    stop-first
ContainerSpec:
 Image:		alpine:latest@sha256:4edbd2beb5f78b1014028f4fbb99f3237d9561100b6881aabbf5acce2c4f9454
 Args:		ping docker.com
 Init:		false
Resources:
Endpoint Mode:	vip
```

> 提示：要以 json 格式返回服务详细信息，请运行不带标志的相同命令。`--pretty`

```bash
$ docker service inspect helloworld
[
    {
        "ID": "ktcjndh4nj4nbwp461idnob0y",
        "Version": {
            "Index": 28
        },
        "CreatedAt": "2022-04-05T15:00:31.72873489Z",
        "UpdatedAt": "2022-04-05T15:00:31.72873489Z",
        "Spec": {
            "Name": "helloworld",
            "Labels": {},
            "TaskTemplate": {
                "ContainerSpec": {
                    "Image": "alpine:latest@sha256:4edbd2beb5f78b1014028f4fbb99f3237d9561100b6881aabbf5acce2c4f9454",
                    "Args": [
                        "ping",
                        "docker.com"
                    ],
                    "Init": false,
                    "StopGracePeriod": 10000000000,
                    "DNSConfig": {},
                    "Isolation": "default"
                },
                "Resources": {
                    "Limits": {},
                    "Reservations": {}
                },
                "RestartPolicy": {
                    "Condition": "any",
                    "Delay": 5000000000,
                    "MaxAttempts": 0
                },
                "Placement": {
                    "Platforms": [
                        {
                            "Architecture": "amd64",
                            "OS": "linux"
                        },
                        {
                            "OS": "linux"
                        },
                        {
                            "OS": "linux"
                        },
                        {
                            "Architecture": "arm64",
                            "OS": "linux"
                        },
                        {
                            "Architecture": "386",
                            "OS": "linux"
                        },
                        {
                            "Architecture": "ppc64le",
                            "OS": "linux"
                        },
                        {
                            "Architecture": "s390x",
                            "OS": "linux"
                        }
                    ]
                },
                "ForceUpdate": 0,
                "Runtime": "container"
            },
            "Mode": {
                "Replicated": {
                    "Replicas": 1
                }
            },
            "UpdateConfig": {
                "Parallelism": 1,
                "FailureAction": "pause",
                "Monitor": 5000000000,
                "MaxFailureRatio": 0,
                "Order": "stop-first"
            },
            "RollbackConfig": {
                "Parallelism": 1,
                "FailureAction": "pause",
                "Monitor": 5000000000,
                "MaxFailureRatio": 0,
                "Order": "stop-first"
            },
            "EndpointSpec": {
                "Mode": "vip"
            }
        },
        "Endpoint": {
            "Spec": {}
        }
    }
]
```

- ##### 查看哪些节点正在运行服务

```
$ docker service ps helloworld
ID             NAME           IMAGE           NODE      DESIRED STATE   CURRENT STATE            ERROR     PORTS
3s1myvuaoc0k   helloworld.1   alpine:latest   8886      Running         Running 31 minutes ago
```

- ##### 查看有关任务容器的详细信息

需要在运行任务的节点上运行：

```bash
$ docker ps
CONTAINER ID   IMAGE           COMMAND             CREATED          STATUS          PORTS     NAMES
3128610843c3   alpine:latest   "ping docker.com"   38 minutes ago   Up 38 minutes             helloworld.1.3s1myvuaoc0kdvlaz4fgz3usk
```

#### 扩展服务

- ##### 修改正在运行服务的副本数量

```bash
$ docker service scale helloworld=5
helloworld scaled to 5
```

- ##### 查看指定服务的所有副本

```bash
$ docker service ps helloworld
ID             NAME           IMAGE           NODE      DESIRED STATE   CURRENT STATE                ERROR     PORTS
3s1myvuaoc0k   helloworld.1   alpine:latest   8886      Running         Running 43 minutes ago
sub0vfkz17t1   helloworld.2   alpine:latest   88235     Running         Running 53 seconds ago
qiiwkq8eaw4c   helloworld.3   alpine:latest   88236     Running         Running about a minute ago
272jo44c63qd   helloworld.4   alpine:latest   88236     Running         Running about a minute ago
vx61pv6vzm9y   helloworld.5   alpine:latest   8886      Running         Running 2 minutes ago
```

你可以看到，swarm 已经创建了 4 个新的任务，可以扩展到共计 5 个运行 Alpine Linux 的实例。任务分布在群体的三个节点之间。

#### 删除服务

- ##### 删除指定服务

```bash
$ docker service rm helloworld
```

#### 滚动更新

```bash
# 创建一个redis服务，并配置10s的更新延迟
$ docker service create \
  --replicas 3 \
  --name redis \
  --update-delay 10s \
  redis:3.0.6
pvud31bgvl7e5ljf6xuxcn7dh

# 检查服务：redis
$ docker service inspect --pretty redis

ID:		pvud31bgvl7e5ljf6xuxcn7dh
Name:		redis
Service Mode:	Replicated
 Replicas:	3
Placement:
UpdateConfig:
 Parallelism:	1
 Delay:		10s
 On failure:	pause
 Monitoring Period: 5s
 Max failure ratio: 0
 Update order:      stop-first
RollbackConfig:
 Parallelism:	1
 On failure:	pause
 Monitoring Period: 5s
 Max failure ratio: 0
 Rollback order:    stop-first
ContainerSpec:
 Image:		redis:3.0.6@sha256:6a692a76c2081888b589e26e6ec835743119fe453d67ecf03df7de5b73d69842
 Init:		false
Resources:
Endpoint Mode:	vip

# 更新容器镜像，升级redis版本
docker service update --image redis:3.0.7 redis
redis

# 再次检查redis服务
[root@8886 ~]# docker service inspect --pretty redis

ID:		pvud31bgvl7e5ljf6xuxcn7dh
Name:		redis
Service Mode:	Replicated
 Replicas:	3
Placement:
UpdateConfig:
 Parallelism:	1
 Delay:		10s
 On failure:	pause
 Monitoring Period: 5s
 Max failure ratio: 0
 Update order:      stop-first
RollbackConfig:
 Parallelism:	1
 On failure:	pause
 Monitoring Period: 5s
 Max failure ratio: 0
 Rollback order:    stop-first
ContainerSpec:
 Image:		redis:3.0.7@sha256:730b765df9fe96af414da64a2b67f3a5f70b8fd13a31e5096fee4807ed802e20
 Init:		false
Resources:
Endpoint Mode:	vip

# 查看服务信息
$ docker service ps redis
ID             NAME          IMAGE         NODE      DESIRED STATE   CURRENT STATE            ERROR     PORTS
s3uwdrtssguv   redis.1       redis:3.0.7   88236     Running         Running 3 minutes ago
9nav9b094si0    \_ redis.1   redis:3.0.6   88236     Shutdown        Shutdown 5 minutes ago
tdy6tqfhweo4   redis.2       redis:3.0.7   88235     Running         Running 2 minutes ago
t9vr0kt1h8av    \_ redis.2   redis:3.0.6   88235     Shutdown        Shutdown 3 minutes ago
xjc6v95dayhv   redis.3       redis:3.0.7   8886      Running         Running 3 minutes ago
biy701m9tvat    \_ redis.3   redis:3.0.6   8886      Shutdown        Shutdown 4 minutes ago
```

#### 启动、停止服务

```bash
docker stack deploy $stack_name -c docker-compose.yml -c other.yml ...
```

`stack_name` 是 Stack 名称。可以用`-c`指定多个`docker-compose.yml`文件，也可以相同 Stack 下分批次 deploy 多个文件。

需要停止 Stack 的所有服务时，可以执行以下命令：

```bash
docker stack rm $stack_name
```
