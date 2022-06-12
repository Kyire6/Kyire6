---
title: JVM探究
tags:
  - 笔记
  - 技巧
categories: Java
cover: https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20211212181003.png
abbrlink: eb9166f8
date: 2021-05-06 16:25:44
updated: 2021-05-06 16:26:26
---
# JVM探究

- 请你谈谈你对JVM的理解，Java8虚拟机和之前的变化更新？
- 什么是OOM，什么是栈溢出StackOverFlowError？怎么分析？
- JVM常用调优参数有哪些？
- 内存快照如何抓取，怎么分析Dump文件？
- 谈谈JVM中，类加载器你的认识？

## JVM的位置

![JVM图解](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210501155102.png)

## JVM的体系结构

![image-20210502005237430](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210502005237.png)

百分之99的JVM调优都是在堆中调优，Java栈、本地方法栈、程序计数器是不会有垃圾存在的。

## 类加载器

作用：加载Class文件~

![在这里插入图片描述](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210501161637.png)

- 虚拟机自带的加载器
- 启动类（根）加载器
- 扩展类加载器
- 应用程序（系统类）加载器

## 双清委派机制

![img](https://img-blog.csdnimg.cn/20201217213314510.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NvZGV5YW5iYW8=,size_16,color_FFFFFF,t_70)

## 沙箱安全机制

Java安全模型的核心就是Java沙箱(sandbox) 

什么是沙箱?沙箱是一个限制程序运行的环境。沙箱机制就是将Java代码限定在虚拟机(JVM)特定的运行范围中，并且严格限制代码对本地系统资源访问，通过这样的措施来保证对代码的有效隔离，防止对本地系统造成破坏。

沙箱主要限制系统资源访问，那系统资源包括什么? CPU、内存、文件系统、网络。不同级别的沙箱对这些资源访问的限制也可以不一样。

所有的Java程序运行都可以指定沙箱，可以定制安全策略。

在Java中将执行程序分成本地代码和远程代码两种，本地代码默认视为可信任的，而远程代码则被看作是不受信的。对于授信的本地代码,可以访问一切本地资源。而对于非授信的远程代码在早期的Java实现中，安全依赖于沙箱Sandbox)机制。如下图所示JDK1.0安全模型

![img](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210501163417.png)

但如此严格的安全机制也给程序的功能扩展带来障碍，比如当用户希望远程代码访问本地系统的文件时候，就无法实现。因此在后续的Java1.1版本中，针对安全机制做了改进，增加了安全策略，允许用户指定代码对本地资源的访问权限。如下图所示JDK1.1安全模型

![在这里插入图片描述](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210501164943.png)

在Java1.2版本中，再次改进了安全机制，增加了代码签名。不论本地代码或是远程代码，都会按照用户的安全策略设定，由类加载器加载到虚拟机中权限不同的运行空间，来实现差异化的代码执行权限控制。如下图所示

![在这里插入图片描述](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210501164956.png)

当前最新的安全机制实现，则引入了域(Domain)的概念。虚拟机会把所有代码加载到不同的系统域和应用域,系统域部分专门负责与关键资源进行交互，而各个应用域部分则通过系统域的部分代理来对各种需要的资源进行访问。虚拟机中不同的受保护域(Protected Domain),对应不一样的权限(Permission)。存在于不同域中的类文件就具有了当前域的全部权限，如下图所示最新的安全模型(jdk 1.6)

![在这里插入图片描述](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210501165007.png)

**组成沙箱的基本条件**

- 字节码校验器(bytecode verifier) :确保Java类文件遵循Java语言规范。这样可以帮助Java程序实现内存保护。但并不是所有的类文件都会经过字节码校验，比如核心类。
- 类裝载器(class loader) :其中类装载器在3个方面对Java沙箱起作用
  - 它防止恶意代码去干涉善意的代码;
  - 它守护了被信任的类库边界;
  - 它将代码归入保护域,确定了代码可以进行哪些操作；

虚拟机为不同的类加载器载入的类提供不同的命名空间，命名空间由一系列唯一的名称组成， 每一个被装载的类将有一个名字，这个命名空间是由Java虚拟机为每一个类装载器维护的，它们互相之间甚至不可见。

类装载器采用的机制是双亲委派模式。

1. 从最内层JVM自带类加载器开始加载,外层恶意同名类得不到加载从而无法使用;
2. 由于严格通过包来区分了访问域,外层恶意的类通过内置代码也无法获得权限访问到内层类，破坏代码就自然无法生效。

- 存取控制器(access controller) :存取控制器可以控制核心API对操作系统的存取权限，而这个控制的策略设定,可以由用户指定。
- 安全管理器(security manager) : 是核心API和操作系统之间的主要接口。实现权限控制，比存取控制器优先级高。
- 安全软件包(security package) : java.security下的类和扩展包下的类，允许用户为自己的应用增加新的安全特性，包括:
  -  安全提供者 
  - 消息摘要 
  - 数字签名
  - 加密
  - 鉴别

## Native

- native :凡是带了native关键字的，说明java的作用范围达不到了，回去调用底层c语言的库!
- 会进入本地方法栈
- 调用本地方法本地接口 JNI (Java Native Interface)
- JNI作用:开拓Java的使用，融合不同的编程语言为Java所用!最初: C、C++
- Java诞生的时候C、C++横行，想要立足，必须要有调用C、C++的程序
- 它在内存区域中专门开辟了一块标记区域: Native Method Stack，登记native方法
- 在最终执行的时候，加载本地方法库中的方法通过JNI

### **Native Method Stack**

它的具体做法是Native Method Stack中登记native方法，在( Execution Engine )执行引擎执行的时候加载Native Libraies。[本地库]

### **Native Interface本地接口**

本地接口的作用是融合不同的编程语言为Java所用，它的初衷是融合C/C++程序, Java在诞生的时候是C/C++横行的时候，想要立足，必须有调用C、C++的程序，于是就在内存中专门开辟了块区域处理标记为native的代码，它的具体做法是在Native Method Stack 中登记native方法,在( Execution Engine )执行引擎执行的时候加载Native Libraies。

目前该方法使用的越来越少了，除非是与硬件有关的应用，比如通过Java程序驱动打印机或者Java系统管理生产设备，在企业级应用中已经比较少见。因为现在的异构领域间通信很发达，比如可以使用Socket通信,也可以使用Web Service等等，不多做介绍!

## PC寄存器

程序计数器: Program Counter Register

每个线程都有一个程序计数器，是线程私有的，就是一个指针, 指向方法区中的方法字节码(用来存储指向像一条指令的地址， 也即将要执行的指令代码)，在执行引擎读取下一条指令, 是一个非常小的内存空间，几乎可以忽略不计

## 方法区

方法区是被所有线程共享,所有字段和方法字节码，以及一些特殊方法，如构造函数,接口代码也在此定义,简单说，所有定义的方法的信息都保存在该区域,此区域属于共享区间;

==静态变量、常量、类信息(构造方法、接口定义)、运行时的常量池存在方法区中，但是实例变量存在堆内存中，和方法区无关==

## 栈

栈:先进后出

桶:后进先出

队列:先进先出( FIFO : First Input First Output )

栈：**栈内存，主管程序的运行，生命周期和线程同步**

**线程结束，栈内存也就是释放，对于栈来说，不存在垃圾回收问题**

一旦线程结束，栈就Over!

栈内存中:

**8大基本类型+对象引用+实例的方法**

**栈运行原理:栈帧**

栈满了: StackOverflowError

![在这里插入图片描述](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210501173518.png)

![在这里插入图片描述](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210501173432.png)

## 三种JVM

- **HotSpot VM**

  ```
  HotSpot VM的热点代码探测能力可以通过执行计数器找出最具有编译价值的代
  码，然后通知JIT编译器以方法为单位进行编译。 如果一个方法被频繁调用，或方法中有效
  循环次数很多，将会分别触发标准编译和OSR（栈上替换）编译动作。 通过编译器与解释器
  恰当地协同工作，可以在最优化的程序响应时间与最佳执行性能中取得平衡，而且无须等待
  本地代码输出才能执行程序，即时编译的时间压力也相对减小，这样有助于引入更多的代码
  优化技术，输出质量更高的本地代码
  ```

- **JRockit**

  ```
  1.JRockit VM曾经号称“世界上速度最快的Java虚拟机”
  2.由于专注于服务器端应用，它可以不太关注程序启动速度，因此JRockit内部不包含解析器实现，全部代码都靠即时
  编译器编译后执行。 除此之外，JRockit的垃圾收集器和MissionControl服务套件等部分的实
  现，在众多Java虚拟机中也一直处于领先水平
  ```

- **J9**

  ```
  1.IBM J9 VM并不是IBM公司唯一的Java虚拟机，不过是目前其主力发展的Java虚拟机，IBM J9 VM原本是内部开发代号，
  正式名称是“IBM Technology for Java Virtual Machine”，简称IT4J，只是这个名字太拗口了一点，普及程度不如J9.
  2.与BEA JRockit专注于服务器端应用不同，IBM J9的市场定位与Sun HotSpot比较接近，它是一款设计上从服务器端
  到桌面应用再到嵌入式都全面考虑的多用途虚拟机***，J9的开发目的是作为IBM公司各种Java产品的执行平台，它的主
  要市场是和IBM产品（如IBM WebSphere等）搭配以及在IBM AIX和z/OS这些平台上部署Java
  应用。
  ```

## 堆

Heap, 一个JVM只有一个堆内存，堆内存的大小是可以调节的。

类加载器读取了类文件后，一般会把什么东西放到堆中?

类, 方法，常量,变量~，保存我们所有引用类型的真实对象;

堆内存中还要细分为三个区域:

- 新生区(伊甸园区) Young/New
- 老年区old
- 永久区Perm

![img](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210501174842.png)

GC垃圾回收,主要是在伊甸园区和养老区~

假设内存满了,OOM,堆内存不够! java.lang.OutOfMemoryError:Java heap space

永久存储区里存放的都是Java自带的 例如lang包中的类 如果不存在这些，Java就跑不起来了

在JDK8以后，永久存储区改了个名字(元空间)

[参考链接](https://www.cnblogs.com/duanxz/p/3726574.html)

## 新生区、老年区

- 对象：诞生和成长的地方，甚至死亡;
- 伊甸园，所有的对象都是在伊甸园区new出来的!
- 幸存者区(0,1)

![在这里插入图片描述](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210501175400.png)

**伊甸园满了就触发轻GC，经过轻GC存活下来的就到了幸存者区，幸存者区满之后意味着新生区也满了，则触发重GC，经过重GC之后存活下来的就到了养老区。**

真理:经过研究，99%的对象都是临时对象!|

[参考链接](https://www.cnblogs.com/duanxz/p/3726574.html)

## 永久区

这个区域常驻内存的。用来存放JDK自身携带的Class对象。Interface元数据，存储的是Java运行时的一些环境~ 这个区域不存在垃圾回收，关闭虚拟机就会释放内存

- jdk1.6之前：永久代,常量池是在方法区;
- jdk1.7：永久代,但是慢慢的退化了，去永久代，常量池在堆中
- jdk1.8之后：无永久代,常量池在元空间

![在这里插入图片描述](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210501180058.png)

元空间：逻辑上存在，物理上不存在 (因为存储在本地磁盘内) 所以最后并不算在JVM虚拟机内存中

## 堆内存调优

测试代码：

```java
public static void main(String[] args) {
    String s = "";
    while (true) {
        s += "11111111111111111111111111111111111111111111111111111";
    }
}
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200715152713345.png#pic_center)

![在这里插入图片描述](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210501190206.png)

**在一个项目中，突然出现了OOM故障,那么该如何排除 研究为什么出错~**

- 能够看到代码第几行出错:内存快照分析工具，MAT, Jprofiler
- Debug, 一行行分析代码!

**MAT, Jprofiler作用**

- 分析Dump内存文件,快速定位内存泄露;
- 获得堆中的数据
- 获得大的对象~

**Jprofile使用**

1. 在idea中下载jprofile插件
2. 联网下载jprofile客户端
3. 在idea中VM参数中写参数 -Xms1m -Xmx8m -XX: +HeapDumpOnOutOfMemoryError
4. 运行程序后在jprofile客户端中打开找到错误 告诉哪个位置报错

**命令参数详解**

-  -Xms设置初始化内存分配大小/164
-  -Xmx设置最大分配内存，默以1/4
-  -XX: +PrintGCDetails // 打印GC垃圾回收信息
-  -XX: +HeapDumpOnOutOfMemoryError //oom **DUMP**

## GC

![在这里插入图片描述](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210501191953.png)

JVM在进行GC时，并不是对这三个区域统一回收。 大部分时候，回收都是新生代~

- 新生代
- **幸存区(form，to)**
- 老年区

GC两种类:轻GC (普通的GC)， 重GC (全局GC)

GC常见面试题目:

- **JVM的内存模型和分区~详细到每个区放什么?**

  ![在这里插入图片描述](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210501192059.png)

- **堆里面的分区有哪些?**

  Eden, form, to, 老年区,说说他们的特点!

- **GC的算法有哪些?**

  标记清除法，标记整理,复制算法，引用计数器

- **轻GC和重GC分别在什么时候发生?**

[参考链接](https://www.cnblogs.com/qianguyihao/p/4744233.html)
