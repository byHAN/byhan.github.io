---
layout: post
title: Linux 进程状态
category: 技术
tags: 
keywords:
description: 
---


STAT狀態位常見的狀態字符
D 无法中断的休眠状态（通常 IO 的进程）；
R 正在运行可中在队列中可过行的；
S 处于休眠状态；
T 停止或被追踪；
W 进入内存交换  （从内核2.6开始无效）；
X 死掉的进程   （基本很少見）；
Z 僵尸进程；
< 优先级高的进程
N 优先级较低的进程
L 有些页被锁进内存；
s 进程的领导者（在它之下有子进程）；
l 多进程的（使用 CLONE_THREAD, 类似 NPTL pthreads）；
+ 位于后台的进程组；