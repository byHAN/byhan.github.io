---
layout: post
title: virsh命令（1）secret,snapshot,pool,volume部分
category: 技术
tags: 虚拟化层
keywords: 
description: 
---

#### Secret (help keyword 'secret'): ####

**secret-define**:                  define or modify a secret from an XML file(根据包含sercret属性XML创建一个虚拟机) 
**secret-dumpxml**:                 secret attributes in XML(显示XML中sercret属性的配置)  
**secret-get-value**:               Output a secret value(获取一个secret属性的值)  
**secret-list**:                    list secrets(显示sercret列表)  
**secret-set-value**:               set a secret value(设置一个secret属性的值)  
**secret-undefine**:                undefine a secret(删除一个secret-define的)  
![](http://i.imgur.com/V59Lfef.png)


**Snapshot (help keyword 'snapshot'):**

snapshot-create:                Create a snapshot from XML(根据XML定义创建一个快照文件)  
snapshot-create-as:             Create a snapshot from a set of args(	根据配置参数创建一个快照文件)  
snapshot-current:               Get or set the current snapshot(获取或者设置当前的快照)  
snapshot-delete:                Delete a domain snapshot(删除虚拟机快照)  
snapshot-dumpxml:               Dump XML for a domain snapshot(dump出一个虚拟机快照的XML信息)  
snapshot-edit:                  edit XML for a snapshot(修改快照的XML定义)  
snapshot-info:                  snapshot information(快照信息)  
snapshot-list:                  List snapshots for a domain(显示一个虚拟机的快照信息)  
snapshot-parent:                Get the name of the parent of a snapshot(获取一个快照文件的父名称信息)  
snapshot-revert:                Revert a domain to a snapshot(从快照文件恢复一个虚拟机)  

**Storage Pool (help keyword 'pool'):**

find-storage-pool-sources-as:   find potential storage pool sources(查找潜在的存储池资源)  
find-storage-pool-sources:      discover potential storage pool sources(发现潜在的存储池资源)  
pool-autostart:                 autostart a pool(设置资源池自动启动)  
pool-build:                     build a pool(创建一个资源池)  
pool-create-as:                 create a pool from a set of args(根据提供的参数创建并启动一个资源池)  
pool-create:                    create a pool from an XML file(根据XML定义创建并启动一个资源池)  
pool-define-as:                 define a pool from a set of args(根据设置的参数创建一个资源池)  
pool-define:                    define an inactive persistent storage pool or modify an existing persistent one from an XML file(创建一个资源池)  
pool-delete:                    delete a pool  
pool-destroy:                   destroy (stop) a pool(删除一个资源池)  
pool-dumpxml:                   pool information in XML(显示资源池的XML定义)  
pool-edit:                      edit XML configuration for a storage pool(修改一个存储资源池的XML定义) 
pool-info:                      storage pool information(存储资源池信息显示)  
pool-list:                      list pools(显示资源池列表)  
pool-name:                      convert a pool UUID to pool name(根据资源池的UUID获取资源池名称)  
pool-refresh:                   refresh a pool(刷新一个资源池)  
pool-start:                     start a (previously defined) inactive pool(启动一个inactive状态的资源池)  
pool-undefine:                  undefine an inactive pool(删除一个inactive状态的资源池)  
pool-uuid:                      convert a pool name to pool UUID(根据资源池的名称获取其UUID)  

**Storage Volume (help keyword 'volume'):**

vol-clone                      clone a volume.(克隆一个卷) 
vol-create-as                  create a volume from a set of args(根据设定的参数克隆一个卷)  
vol-create                     create a vol from an XML file(根据XML定义克隆一个卷)  
vol-create-from                create a vol, using another volume as input(	用另外一个卷作为输入，克隆一个卷)  
vol-delete                     delete a vol(删除一个卷)  
vol-download                   download volume contents to a file(将卷的内容下载到一个文件里面)  
vol-dumpxml                    vol information in XML(显示卷的XML定义)  
vol-info                       storage vol information(卷的详细信息)  
vol-key                        returns the volume key for a given volume name or path(根据卷的名字或存储路径获取卷的key值)  
vol-list                       list vols(显示所有卷列表)  
vol-name                       returns the volume name for a given volume key or path(根据卷的key值或存储路径获取卷的名字)  
vol-path                       returns the volume path for a given volume name or key(根据卷的key值或名称获取卷的存储路径)  
vol-pool                       returns the storage pool for a given volume key or path(根据卷的key值或存储路径，获取卷的存储资源池)  
vol-resize                     resize a vol(为一个卷扩容)  
vol-upload                     upload file contents to a volume(将文件内容upload到卷里面)  
vol-wipe                       wipe a vol(擦除卷里面的数据)  