---
layout: post
title: Linux网桥
category: 技术
tags: 技术
keywords:
description: 
---

**1.桥**

	DEVICE="br0"
	BOOTPROTO="static"
	IPADDR=192.168.2.207
	NETMASK=255.255.255.0
	GATEWAY=192.168.2.1
	IPV6INIT=no
	ONBOOT="yes"
	TYPE="Bridge"


**2.上行链路**

	DEVICE=enp2s0f0
	TYPE=Ethernet
	BRIDGE=br0
	ONBOOT=yes
	BOOTPROTO=none