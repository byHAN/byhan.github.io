---
layout: post
title: 向openstack社区合入代码记
category: 技术
tags: openstack虚拟机创建流程
keywords: 
description: 
---


尝试合入代码，按照社区指导文档配置，在git review步骤死活不行。
最后通过资料，git review加上参数-v查看详情，发现默认情况下ssh 通过29418口连接gerrit
20160325090959

然而，据说29418被上国给屏蔽了，呵呵哒
 
# 解决方法 #

参考社区指导文档 的步骤

    Accessing Gerrit over HTTPS

使用https 的方式登录，此外需要在review.openstack.org网站上配置HTTP password in Gerrit


另外，可能需要commit-msg文件上传至/usr/share/git-core/templates/hooks目录

可以把下面保存为commit-msg