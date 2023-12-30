---
title: nginx学习
date: 2022-01-19 09:26:25
description: nginx
tags: nginx
---

### 1. 基础知识

​	systemctl restart network 重启网卡

​	ip addr 查看网络信息

#### 1.1 nginx全貌

![image-20220424213429884](C:\Users\11655\AppData\Roaming\Typora\typora-user-images\image-20220424213429884.png)

#### 1.2 nginx的版本

![image-20220424215441104](C:\Users\11655\AppData\Roaming\Typora\typora-user-images\image-20220424215441104.png)

### 1.3 nginx的优势

1. 使用多路IO复用
2. 使用EPOLL模型
3. 与CPU的亲和

​		是一种把CPU核心和nginx工作进程绑定方式，把每个worker进程固定在一个cpu上执行，减少切换cpu的cache miss，获得更好的性能。

​	4. sendfile
