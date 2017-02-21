---
layout: post
title: openstack-horizon 证书认证
category: 技术
tags: openstack
keywords: 
description: 
---

此文章参考这篇文章
验证环境为2016.04 devstack环境

参考文档：

[http://www.cacert.org/](http://www.cacert.org/)
[http://docs.openstack.org/juno/config-reference/content/configure-dashboard.html](http://docs.openstack.org/juno/config-reference/content/configure-dashboard.html)
[http://blog.sina.com.cn/s/blog_6de3aa8a0101osdd.html](http://blog.sina.com.cn/s/blog_6de3aa8a0101osdd.html)


整体上的流程参考新浪上的zhangguoqing同学的文章即可

现将相关差异点整理如下：
路径

当前相关的配置文件已经做了规整，换句话说，当前配置文件修改这里即可

![](http://i.imgur.com/rD87qdK.png)

内容

主要参考官网上的内容，把相关信息替换即可

![](http://i.imgur.com/8B2OXbe.png)

注意问题

重启apache的可能会报不识别SSL命令，如下图
需要加载下ssl相关内容，具体按照下面错误信息去谷歌，不记得了（o(╯□╰)o）
