---
title: Git分支管理
tags:
  - 笔记
  - Git
categories: Linux
cover: https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20211212175556.png
abbrlink: 13a54546
date: 2021-04-05 16:25:44
updated: 2021-04-05 16:26:26
---
# Git 分支管理
## 什么是分支

在版本控制过程中，同时推进多个任务，为每个任务，我们就可以创建每个任务单独分支。使用分支意味着程序员可以把自己的工作从开发主线上分离开来，开发自己分支的时候，不会影响主线分支的运行。对于初学者而言，分支可以简单理解为副本，一个分支就是一个单独的副本。（分支底层其实也是指针的引用）

![image-20220226151832730](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220226151839.png)	

## 分支的好处

同时并行推进多个功能开发，提高开发效率。

各个分支在开发过程中，如果某一个分支开发失败，不会对其他分支有任何影响。失败的分支删除重新开始即可。

## 分支的操作

| 命令名称            | 作用                         |
| ------------------- | ---------------------------- |
| git branch 分支名   | 创建分支                     |
| git branch -v       | 查看分支                     |
| git checkout 分支名 | 切换分支                     |
| git merge 分支名    | 把指定的分支合并到当前分支上 |

