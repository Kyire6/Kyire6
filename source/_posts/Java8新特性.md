---
title: Java8新特性
tags:
  - 笔记
  - 技巧
categories: Java
cover: 'https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20211212173613.png'
abbrlink: 397c083a
date: 2021-06-17 16:25:44
updated: 2021-06-17 16:26:26
---
# Java8新特性

## 前言

Java8 有一个非常显著的特点，就是提供了函数式编程，本文也将介绍 Java8 的几个新特性来实现函数式编程。学会这些 API 能够编写出简单、干净、易读的代码（尤其是对集合的操作）。Java8 新特性包括：

- Lambda 表达式
- Stream API
- 接口新特性
  - 函数式接口（`@FunctionalInterface`）
  - 默认方法（`default`）
- Optional API
- 方法引用

## 一、Lambda 表达式

Lambda 表达式，也可称为闭包，是 Java8 发布的一个新特性。

Lambda 允许把函数作为一个方法的参数（将函数传递到方法中）。

使用 Lambda 表达式可以使代码更简洁紧凑。

**下面是 Lambda 的主要特征：**

- 可选类型声明：不需要声明参数类型，编译器可以统一识别参数值
- 可选的参数圆括号：一个参数无需定义圆括号，但多个参数需要定义圆括号
- 可选的大括号：如果主体包含了一个语句，就不需要使用大括号
- 可选的返回关键字：如果主体只有一个表达式返回值则编译器会自动返回值，大括号需要指定明表达式返回了一个数值

**Lambda 表达式的简单例子：**

```java
// 1. 不需要参数,返回值为 5  
() -> 5  
  
// 2. 接收一个参数(数字类型),返回其2倍的值  
x -> 2 * x  
  
// 3. 接受2个参数(数字),并返回他们的差值  
(x, y) -> x – y  
  
// 4. 接收2个int型整数,返回他们的和  
(int x, int y) -> x + y  
  
// 5. 接受一个 string 对象,并在控制台打印,不返回任何值(看起来像是返回void)  
(String s) -> System.out.print(s)
```

**变量作用域**

如果你曾使用过匿名内部类，也许会遇到这种情况：需要引用它所在方法中的变量。这个变量必须声明为 `final` 类型，将变量声明为 final ，意味着不能为其重复赋值。Java8 虽然放松了这一限制，可以引用非 final 变量，但是该变量隐性为 final 变量。虽然无需声明 final 类型，但在 Lambda 表达式中，若重复为其赋值，编译器就会报错。

## 二、Stream API 

Stream（流）是一个来自数据源的元素队列并支持聚合操作

- 元素是特定类型的对象，形成一个队列。Java 中的 Stream 并不会存储元素，而是按需计算
- **数据源** 流的来源。可以是集合，数组，IO channel，产生器 generate 等
- **聚合操作** 类似 SQL 一样的操作，比如 filter、map、reduce、find、match、sorted等

和以前的 Collection 操作不同，Stream 操作还有两个基础的特征：

- **pipelining**：中间操作都会返回流对象本身，这样多个操作可以串联成一个管道，如同流式风格（fluent style）。这样做可以对操作进行优化
- **内部迭代：**以前对集合遍历都是通过 Iterator 或者 for-each 的方式，显示的在集合外部进行迭代，这叫做外部迭代。Stream 提供了内部迭代的方式。

