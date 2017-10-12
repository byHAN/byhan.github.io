---
layout: post
title: 重做ubuntu内核
category: 技术
tags: 
keywords: change kernel version ubuntu
description: 
---

## 下载内核 ##

    git clone git://kernel.ubuntu.com/ubuntu/ubuntu-trusty.git

## 上传代码 ##

这里把代码上传到gitlab代为托管

修改remote的url  

    git@10.121.5.37:virt/ubuntu-trusty.git

注意：上传量交大，这里使用git的方式（可以配置密钥免认证），因为用http的方式传输会端口

    git push origin --all
    git push origin --tags

## 构建编译环境 ##

### 准备 ###

安装[参考文档](https://wiki.ubuntu.com/Kernel/BuildYourOwnKernel)构建编译环境

由于需要针对3.16.0-37.51~14.04.1这个版本，回合bug

这里需要用tag构建一个分支,假设分支为fh

    git branch fh Ubuntu-lts-3.16.0-37.51_14.04.1
    git checkout fh

修复bug(本文背景是回合了一个内核bug)

提交修复到gitlab

    git push origin fh

### 编译 ###

    debian/rules clean
    debian/rules binary-headers binary-generic


如果想改包名的话，修改/debian/changelog即可
如果要修改内核版本号的话，需要修改/debian.utopic/changelog
用dch -i修改即可








## 参考文献 ##

[https://wiki.ubuntu.com/Kernel/BuildYourOwnKernel](https://wiki.ubuntu.com/Kernel/BuildYourOwnKernel)