---
layout: post
title: openstack wiki
category: 技术
tags: openstack
keywords: 
description: 
---

#### 配置驱动器(configuration drive) ####

创建虚拟机的一个高级选项。
如果选择了这个选项，OpenStack 把元数据写到一个只读的配置驱动器中，在引导时会被附加到实例中（而不是加入到 Compute 的元数据服务中）。在实例被引导后，您可以挂载这个驱动器来查看其中的内容（您可以为实例提供文件）。
#### 定制脚本源 ####

可以提供一组命令或一个脚本文件，当实例被引导后运行它们。例如，使用它们来设置实例的主机名 ，或设置一个用户密码。如果选择了 '直接输入'，您可以在脚本数据项中直接输入您的命令；如果使用其它选项，则指定您的脚本文件。请注意：任何以 '#cloud-config' 开始的脚本都会被解析为使用 cloud-config 语法（如需了解与此语法相关的信息，请参阅 http://cloudinit.readthedocs.org/en/latest/topics/examples.html

#### 主机集合（Host aggregate） ####

一个主机集合就是 OpenStack 部署中的一个逻辑组，它包括了一组 Compute 主机，以及相关的元数据。一个主机可以属于多个主机集合。只有系统管理员可以查看或创建主机集合。
#### 可用域（Availability zone） ####

可用域就是最终用户可以看到的主机集合。最终用户无法看到这个域是由哪些主机组成的，也无法看到这个域的元数据，而只能看到域的名称。