---
layout: post
title: make patch for libvirt in centos
category: 技术
tags: 虚拟化层
keywords: 
description: 
---



配置git

git config  user.name "byhan"
git config  user.email 2005hanbaoying@gmail.com
![](http://i.imgur.com/1GstM4D.png)

patch 
不要使用git diff因为这样生成的是标准patch，相关信息没有包含
打包的时候会报错  


我们同样用上面那个例子的工作目录，这次，我们在Fix分支中的a添加了新行之后，用git format-patch生成一个patch。
sweetdum@sweetdum-ASUS:~/GitEx$ git checkout -b Fix
Switched to branch ‘Fix’

修改源码


sweetdum@sweetdum-ASUS:~/GitEx$ git commit -a -m “libvirt-sanlock-enable-rbd”
[Fix 6991743] Fix1
1 files changed, 1 insertions(+), 0 deletions(-)
sweetdum@sweetdum-ASUS:~/GitEx$ git format-patch -M master
0001-Fix1.patch

git format-patch的-M选项表示这个patch要和那个分支比对。现在它生成了一个patch文件，我们看看那是什么：

sweetdum@sweetdum-ASUS:~/GitEx$ cat 0001-Fix1.patch
From 6991743354857c9a6909a253e859e886165b0d90 Mon Sep 17 00:00:00 2001
From: Sweetdumplings <linmx0130@163.com>
Date: Mon, 29 Aug 2011 14:06:12 +0800
Subject: [PATCH] Fix1

—
a |    1 +
1 files changed, 1 insertions(+), 0 deletions(-)

diff –git a/a b/a
index 4add65f..0d295ac 100644
— a/a
+++ b/a
@@ -1 +1,2 @@
This is the file a.
+Fix!!!
—
1.7.4.1

看，这次多了好多东西，不仅有diff的信息，还有提交者，时间等等，仔细一看你会发现，这是个E-mail的文件，你可以直接发送它！这种patch，我们要用git am来应用。