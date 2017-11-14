---
layout: post
title: 调试openstack ut
category: 技术
tags: 
keywords:
description: 
---

    testr list-tests <test_name_regex> > my-list
    python -m testtools.run discover --load-list my-list

## 参考文献 ##

[https://wiki.openstack.org/wiki/Testr](https://wiki.openstack.org/wiki/Testr)