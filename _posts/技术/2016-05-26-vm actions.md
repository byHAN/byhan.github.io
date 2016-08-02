---
layout: post
title: vm actions
category: nova
tags: nova-default
keywords: 
description: 
---

**本文基于[这篇博文](http://www.cnblogs.com/sammyliu/p/4571209.html)整理**

### boot ###

启动一个虚拟机

### delete ###

关闭和删除一个虚拟机

1. dom.destory
2. dom.undefineFlags

### resize-confirm ###

确认resize操作

### resize-revert ###

取消resize操作

### reboot ###

分为soft_boot和hard_boot

#### sof_reboot ####

依赖于guest os及其ACPI。
1. Shutdown domain （domain.shutdown）  
shutdown domain 以后，客户机的磁盘存储处于可用状态，而且该 API 在命令发给 Guest OS 后立即返回  
客户程序需要不断检查其状态来判断是否shutdown 成功  
2. _create_domain
3. Launch domain (domain.createWithFlags)

#### hard_reboot ####

1. Destroy domain （virt_dom.destroy()）  
而 destroy API 执行后，客户机的所有资源都会被归还给 Hypervisor  
（all resources used by it are given back to the hypervisor）  
它的过程是首先给客户机发送一个terminate 指令比如SIGTERM，如果在一定的时间内未成功则直接发送 SIGKILL 指令将虚机杀掉。
2. 根据 domain 的信息，重新生成 domain 的配置 xml
3. 根据 image 重新生成各镜像文件（包括 disk，disk.local 和 disk.swap）
4. 连接各个 volume
5. 将各个 interface 挂到 OVS 上
6. 重新生成和应用 iptales 规则
7. Define domain （conn.defineXML）
8. Launch domain （domain.createWithFlags）

### root-password ###

没有实现

### resize ###

迁移虚拟机或者改变规格（flavor）  

### rebuild ###

重建虚拟机，可以用来撤离虚拟机或者实现HA等  

1. driver.destroy  
（destroy domain，detach volume connections，unplug VIF）
2. driver.spawn

###  image-create ###

快照

### start ###

启动虚拟机  

1. driver._hard_
2.   

### stop ###

停止虚拟机  

1. dom.destroy

### pause ###

暂停虚拟机（数据存储于内存）

1. dom.suspend

### unpause ###

取消暂停

1. dom.resume

### suspend ###

挂起虚拟机（会将虚拟机数据同步到磁盘）

1.  _detach_pci_devices
2.  _detach_sriov_ports
3.  dom.managedSave(0)

### resume ###

恢复挂起 

1. get domain xml
2. _create_domain_and_network
3. attach pci devices
4. attach sriov port

### migrate ###

迁移虚拟机

### injectNetworkInfo ###

注：这个在命令行中没有，在rest接口中有  
Set up basic filtering (MAC, IP, and ARP spoofing protection)  
调用 _conn.nwfilterDefineXML(xml)

### lock ###

 直接在数据库中 instance 上面设置 lock = true

### unlock ###

直接在数据库中 instance 上面设置 lock = false

### backup ###

同 createImage  
可以指定类型（daily 或者 weekly），和保存的 image 的最大数目  
老的image会被删除 {"backup_type": "daily", "rotation": "2", "name": "bk"}  

### live-migration ###

热迁移

### reset-state ###

传入 state 参数，直接修改数据库中 instance 的状态


### get-vnc-console ###

1. 用户查询 VNC 的访问 URL  |
    nova get-vnc-console <server> <console-type>
2. 过 RPC 调用虚机所在的 node 上的 Nova 生成一个 UUID（长度为 4）格式的 token以及格式为 ‘?token=' 的 Access URL。
3. 本地Nova 调用 libvirt driver 从 domain xml 中获取 port
4. 以及从 confi.vncserver_proxyclient_address 获取 host  
    <graphics type='vnc' port='5900' autoport='yes' listen='0.0.0.0' keymap='en-us'>  
    <listen type='address' address='0.0.0.0'/>  
    </graphics>
5. 通过 RPC，调用 consoleauth 的 authorize_console 方法，它将 Access URL， host，port 保存到 memcached。
6. 返回 Access URL 给客户端，比如  

    http://192.168.1.20:6080/vnc_auto.html?token=8dc6f7cb-2e2d-4fbe-abab-3334fe3a19dc  

7. 在客户端浏览器上打开 Access URL，由 Controller 上的 nova-novncproxy 接收并处理
8. nova-novncproxy 通过 RPC 调用 consoleauth.check_token() 方法  
获取 console_type 和 port 等 connect info。期间还会到目的node上去校验这些信息。
9. 连接到目的 node，握手，开始 proxying。

### console-log ###

读取 虚机的 console.log 文件并返回其内容；  
如果没有这文件的话，则使用 “pty”，将其内容写入到 consolue文件并返回其内容。

### restore ###

Restore a previously deleted (but not reclaimed) instance。直接修改数据库。

###  force-delete ###

有 snapshot，则全部删除；然后从 DB 中删除 instance.

### evacuate ###

从 DB 中读取 instance 数据，在一个新的主机上 rebuild。  
它尽可能地将原来的虚机在新的主机上恢复：  
- 虚机的配置：从 DB 中获取，包括 image，block，network 等
- 虚机的数据：如果使用共享存储，则使用共享存储上的虚机数据；如果不使用共享存储，则无法恢复数据
- 内存状态：无法恢复

详细见本人其他博文，[这里](http://www.hanbaoying.com/2016/05/03/openstack-nova-evacuate.html)


### flavor-access-add/flavor-access-remove ###

修改 DB 中flavor表  
新版本，在nova_api这个database下的flavors表中  
不再在nova这个database的instance_types这个表中了（真能折腾）  

###  floating-ip-create/loating-ip-delete ###

分别调用network_api.associate_floating_ip和network_api.disassociate_floating_ip

### fixed-ip-reserve ###

1. network_api.add_fixed_ip_to_instance
2. firewall_driver.setup_basic_filtering

###  fixed-ip-unreserve ###

1. network_api.remove_fixed_ip_from_instance
2. firewall_driver.setup_basic_filtering(instance, nw_info) 

### disassociate_host ###

注：这个在命令行中没有，在rest接口中有  
Associate or disassociate host or project to network。 

1. network_api.associate

###  network-disassociate ###

1. network_api.associate

###  network-associate-host ###

1. network_api.associate

### rescue ###

使用场景是，虚机的启动盘的一个文件被误删除了导致无法再次启动了，或者 admin 的密码忘记了。  
Rescue 功能提供一个解决这类问题的手段。  

1. 保存目前domain 的 xml 配置到 unrescue.xml 文件
2. 根据 image 重新生成启动盘 disk.swap （大小不受 falvor.root_disk_size 控制，尽可能小的一个文件）  
3. 构造一个新的 domain 的 xml 配置，使用 disk.rescue 做启动盘，将原来的 disk 挂载到该 domain，其他的盘和volume不会被挂载。  
4. 将原来的 domain destroy 掉 （virt_dom.destroy）
5. 定义新的 domain （conn.defineXML(xml)）
6. 启动新的 domain （domain.createWithFlags）

### unrescue ###

1. 读取之前保存的 unrescue.xml 文件
2. 将 rescued domain destroy 掉
3. 定义和启动新的domain（同上面5和6）
4. 删除 unrescue.xml 文件

### add-secgroup ###

1. security_group_rpcapi.refresh_security_group_rules

### secgroup-delete ###

1. security_group_rpcapi.refresh_security_group_rules

### shelve ###

当一个虚机不需要使用的时候，可以将其 shelve 起来。  
该操作会创建该虚机的一个快照并传到 Glance 中，然后在 Hypervisor 上将该虚机删除，从而释放其资源。   
其主要过程为：  
1. destroy 虚机 （virt_dom.destroy()）
2. snapshot 该 domain
3. 如果 CONF.shelved_offload_time == 0 的话，将domain 的各种资源（interface，volume，实例文件等）
4. undefine （virt_dom.undefine()）

存在于数据库中，可以通过nova-list查看
存在于glance中，可以通过glance image-list查看

### shelve-offload ###

移除shelve的虚拟机

### unshelve ###

1. 从 DB 中获取 network_info 和 block_d
2. evice_info
2. 从 Glance 中获取 image
3. 象新建一个虚拟那样 spawn 新的虚机
4. 调用 image API 将 image 删除 