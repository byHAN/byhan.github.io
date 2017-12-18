---
layout: post
title: 指定主机创建虚拟机
category: nova
tags: 
keywords: virtio,9p
description: 
---


    nova boot --flavor 123 
    --block-device id=b00326ef-fa27-4470-8458-5757ddae0580,source=image,dest=volume,size=25,bootindex=0 
    --availability-zone nova:jycompute27  
    --nic net-id=ed3fb18f-2862-455d-8fdb-34853e21fb3b --admin-pass Fh123456 --config-drive true test01