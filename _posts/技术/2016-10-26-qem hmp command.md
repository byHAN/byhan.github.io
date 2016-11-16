---
layout: post
title: qemu(2.3)接口梳理
category: 技术
tags: openstack
keywords: multipath call failed exit 1
description: 
---

| qmp | 简介 | hmp | 
| ------| ------ | ------ | 
| qmp_memsave | save to disk virtual memory dump starting at 'addr' of size 'size' | memsave | 
| qmp_inject_nmi | inject an NMI | nmi | 
|qmp_system_reset|reset the system|system_reset|
|qmp_stop|stop emulation|stop|
|qmp_cpu_add|add cpu|cpu-add|
|qmp_memsave|save to disk virtual memory dump starting at 'addr' of size 'size'|memsave|
|qmp_quit|quit the emulator|q/quit(进程退出)|
|qmp_system_powerdown|stop emulation|stop（设置为paused）|
|qmp_cont|resume emulation|c/cont（由paused设置为running状态）|
|qmp_system_powerdown|send system power down event|system_powerdown|
|qmp_system_reset|reset the system|system_reset|
|qmp_system_wakeup|wakeup guest from suspend|system_wakeup|
|||savevm|
||delete a VM snapshot from its tag or id|delvm|
||||
||||
||||
||||
||||
||||
||||
||||
||||


### 部分命令详解 ###

#### cpu-add ####

这个是热插cpu的命令，要使用这个命令需要启动虚拟机的时候在smp选项中带maxcpus参数  
![](http://i.imgur.com/VFAludj.png)

然后执行cpu-add热插入

![](http://i.imgur.com/fvK3fvz.png)


#### memsave ####

格式：  

    memsave addr size file

![](http://i.imgur.com/xhZbRpw.png)  

![](http://i.imgur.com/BWEjyzQ.png)  

然后可以用 hexdump 来校验转储内存数据的正确性  
![](http://i.imgur.com/nHePjWQ.png)  


#### savevm ####

save虚拟机是建立内部快照，需要要求磁盘支持快照，如qcow2  

![](http://i.imgur.com/w1o74pQ.png)