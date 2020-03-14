---
title: cmake学习笔记
date: 2020-02-18 09:12:40
tags:
cover_img:
feature_img:
description:
keywords:
---

## Cmake学习笔记

用于记录一下cmake开发中的一些整理性工作

#### cmake变量

使用 `set(A B)` 来定义`A`这个新的变量，之后就可以通过`${A}`来进行引用，可以想成一个**自定义宏**

#### 动态/静态链接、头文件配置

##### 头文件目录

`INCLUDE_DIRECTORIES`

指定项目使用到的头文件目录，可以一次包含多个头文件目录

##### 添加库文件

``



#### 添加外部文件夹

`add_subdirectory(source_dir,[binary_dir])`

第一个参数就是外部文件夹的位置，而`binary_dir`表示输出的位置。如果代码目录在外部，则必须要指定第二个参数

`ADD_SUBDIRECTORY`