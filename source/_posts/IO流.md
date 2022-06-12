---
title: JAVA-IO流
tags:
  - 笔记
  - Java
categories: Java
cover: https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220106013909.png
abbrlink: 13a54546
date: 2021-04-04 16:25:44
updated: 2021-04-04 16:26:26
---
## 一、前言

Java的I/O主要的用途就是文件数据的读写、数据的网络发送与接收等场合。

流是一组有顺序的，有起点和终点的字节集合，是对数据传输的总称或抽象。即数据在两设备间的传输称之为流，流的本质是数据传输，根据数据传输特性将流抽象为各种类，方便更直观的进行数据操作。对于文件内容的操作主要分为两大类：字符流和字节流。

## 二、I/O流的分类

根据处理数据类型的不同分为：字符流和字节流。

根据数据流向不同分为：输入流和输出流。

### 1、字符流和字节流

字符流的由来：因为数据编码的不同，而有了对字符进行高效操作的流对象。本质其实就是基于字节流流读取时，去查了指定的码表。字节流和字符流的区别：

1. 读写单位不同：字节流以字节（8bit）为单位，字符流以字符为单位，根据码表映射字符，一次可能多个字节。
2. 处理对象不同：字节流能处理所有类型的数据（如图片、avi等），而字符流只能处理字符类型的数据。
3. 字节流在操作的时候本身是不会用到缓冲区的，是文件本事的直接操作的；而字符流在操作的时候下后是会用到缓冲区的，通过缓冲区来操作文件。

**结论：优先选用字节流。首先因为硬盘上的所有文件都是以字节的形式进行传输或者保存的，包括图片等内容。但是字符只是在内存汇总才会形成，所以在开发中，字节流使用广泛。**

### 2、输入流和输出流

对输入流只能进行读操作，对输出流只能进行写操作，程序中需要根据待传输数据的不同特性而使用不同的流。

### 3、常见I/O流对象

- InputStream

  ![img](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210401214602.png)

- OutputStream

  <img src="https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210401214647.png" alt="img"  />

- Reader

  ![img](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210401214855.png)

- Writer

  ![img](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210401214921.png)

## 三、常见问题

### 1、字节流和字符流的使用情况

字符流和字节流的使用范围：字节流一般用来处理图像、视频，以及PPT、Word类型的文件。字符流一般用于处理纯文本类型的文件，如txt文件等，字节流可以用来处理纯文本文件，但字符流不能用户处理图像、视频等非文本类型的文件。

### 2、字符流与字节流的转换

转换流的作用，文本文件在硬盘中以字节流的形式存储时，通过InputStreamReader读取后转化成字符流交给程序处理，程序处理的字符流通过OutputStreamWriter转换成字节流保存。

### 3、字节流与字符流的区别

字节流没有缓冲区，是直接输出的，而字符流需要输出到缓冲区的。因此在输出时，字节流不调用close()方法时，信息已经输出了，而字符流只有在调用close()方法关闭缓冲区时，信息才输出。要想字符流在未关闭时输出信息，需要手动调用flush()方法。

1. 读写单位不同：字节流以字节（8bit）为单位，字符流以字符为单位，根据码表映射字符，一次可能多个字节。
2. 处理对象不同：字节流能处理所有类型的数据（如图片、avi等），而字符流只能处理字符类型的数据。
3. 字节流在操作的时候本身是不会用到缓冲区的，是文件本事的直接操作的；而字符流在操作的时候下后是会用到缓冲区的，通过缓冲区来操作文件。

## 四、IO/NIO面试题

### 1、Java中IO流分为几种

1. 按照流的流向分：分为输入流与输出流
2. 按照操作单元分：分为字节流与字符流
3. 按照流的角色分：分为节点流与处理流

### 2、Java IO与 NIO的区别

NIO即New IO，这个库是在JDK1.4中才引入的。NIO和IO有相同的作用和目的，NIO主要用到的是块，所以NIO的效率要比IO高很多。在Java API中提供了两套NIO，一套是针对标准输入输出NIO，另一套就是网络编程NIO。

### 3、Java中常用的IO类有哪些

```
File
FileInputStream、FileOutputStream
BufferedInputStream、BufferedOutputStream
FileReader、FileWrter
BufferedReader、BufferedWriter
ObjectInputStream、ObjectOutputStream
```

### 4、字节流与字符流的区别

字节流是以字节为单位进行传输，大小为8bit

字符流是以字符为单位进行传输，大小为16bit
