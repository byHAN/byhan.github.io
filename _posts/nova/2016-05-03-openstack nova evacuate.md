---
layout: post
title: openstack nova evacuate
category: 技术
tags: openstack 
keywords: 
description: 
---

# 命令行 #

![](http://i.imgur.com/BcqFTIv.png)

# 源码分析 #

## novaclient命令解析 ##

根据novaclient的处理流程，evacuate操作会对应到python-novaclient\novaclient\v2\shell.py文件中的do_evacuate方法中。
此根据参数<server>找到对应的server对象，然后调用server对象的evacuate方法

![](http://i.imgur.com/myOkiUp.png)

server对象的evacuate方法，组装出body对象，调用通用方法_action

![](http://i.imgur.com/WBIh83a.png)

_action方法获取到虚拟机ID,组装url然后调用rest接口

![](http://i.imgur.com/HHX0Zes.png)

#### nova-api ####

rest服务收到请求后首先读取对应的参数

![](http://i.imgur.com/j4zujl7.png)

admin密码相应处理

![](http://i.imgur.com/8IRQbnF.png)

指定了目标主机，则目标主机需要nova-compute服务运行正常。

![](http://i.imgur.com/yYHEFZW.png)

转而调用compute下api对应的evacuate方法

![](http://i.imgur.com/nDrRh4t.png)

compute/api接收到请求后，判断虚拟机所在的主机nova-compute服务是否正常，如果服务正常不允许进行撤离。

![](http://i.imgur.com/zIufkWd.png)

更新状态，发送消息，记录迁移任务等

![](http://i.imgur.com/ajzvAhU.png)

最后调用conductor的rebuild服务（此部分单独成文，请点击链接）

![](http://i.imgur.com/15F79ob.png)

# 验证 #

- 操作前安装的软件是否消失
- 操作前正在执行的服务是什么情况
- 需要注意卷启动和镜像启动虚拟机有什么差异