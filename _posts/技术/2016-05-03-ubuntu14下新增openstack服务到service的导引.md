---
layout: post
title: ubuntu14下新增openstack服务到service的导引
category: 技术
tags: openstack
keywords: 
description: 
---

# 背景 #

最近在研究openstack的一个新服务（cloudkitty）
关于服务启动，官方指导文档给出如下：

Start the API and processing services:

    cloudkitty-api --config-file /etc/cloudkitty/cloudkitty.conf
    cloudkitty-processor --config-file /etc/cloudkitty/cloudkitty.conf

这样启动服务对于pdb调试确实简单，但是当启动的shell死掉后，对应的服务也就死了。
如何像其他服务一样，整合到service中成为守卫进程，现梳理如下：

# 步骤 #

    

1. /etc/init.d目录下新增服务文件，拷贝已有nova文件，变下名字即可
2. /etc/init目录下新增命令执行的conf文件，拷贝已有nova服务，修改名称，并修改里面的内容为对应服务
3. /usr/bin/目录下增加cloudkitty，拷贝nova服务文件，修改名字，并修改内容到对应客户端

# 验证 #

service  cloudkitty-api start
然后ps确认下就可以了