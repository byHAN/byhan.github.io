---
layout: post
title: dommemstat取不到memory.usage
category: 技术
tags: 虚拟化层
keywords: 
description: 
---

![](http://i.imgur.com/doFqjm7.png)

qemu-monitor-command 3 --pretty '{ "execute": "qom-set","arguments": { "path": "balloon0","property": "guest-stats-polling-interval", "value": 2 } }'

qemu-monitor-command 3 --pretty '{"execute":"qom-get","arguments":{"path":"balloon0","property": "guest-stats"}}'