---
title: Github代码仓库更新
tags:
  - Git
  - 教程
categories: Linux
cover: https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20211212181603.png
abbrlink: d1b54efa
date: 2020-02-18 16:30:26
---


## 前言
最近在学习git，接触github也有段时间了，此篇文章记录一下有关github仓库代码更新的方法。

仓库代码更新步骤非常简单，只需要短短的几行代码就可以将我们的本地代码推送到github上的云
仓库中，推送代码到云端是一个优秀程序员必不可少的技能，这样保存代码较安全不易丢失。

--废话不多说，直接开始更新--
<!-- more -->

## 更新步骤
1. 第一步切换到本地仓库中需要更新的目录下
2. 鼠标右键选择`Git Bash Here`选项，打开Git Bash工具
3. 依次运行下列命令：
* 查看仓库状态
```
$ git status
```
* 更新所有代码
```
$ git add *
```
* 本次推送备注
```
$ git commit -m ""
```
* 将本地仓库的代码更新到github仓库的master分支
```
$ git push origin master
```
4. 刷新github仓库页面，更新完成

## 小结
代码记得勤推送，github仓库勤更新！
