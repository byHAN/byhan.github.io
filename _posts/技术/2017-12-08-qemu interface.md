---
layout: post
title: 扩展qemu接口
category: 技术
tags: 
keywords: 
description: 
---

### 背景 ###

项目需要修改qemu的接口。  
我们知道qemu接口对人的是hmp,对机器的是qmp。

#### 1. hmp-commands.hx ####

    {
    .name   = "fs",
    .args_type  = "value:s",
    .params = "target",
    .help   = "request VM to change its fs path",
    .mhandler.cmd = hmp_fs,
    },
    
    STEXI
    @item fs @var{value}
    @findex fs
    Request VM to change its fs path to @var{value}.
    ETEXI

#### 2.qapi-schema.json  ####

    { 'command': 'fs', 'data': {'value': 'str'} }

### 3.hmp.h ###

    void hmp_fs(Monitor *mon, const QDict *qdict);


#### 4.hmp.c ####

	void hmp_fs(Monitor *mon, const QDict *qdict)
	{
	    const char  *value = qdict_get_str(qdict, "value");
	    Error *err = NULL;
	
	    qmp_fs(value, &err);
	    if (err) {
	        error_report_err(err);
	    }
	}

#### 5.实现qpm接口 ####

	void qmp_fs(const char *target, Error **errp)
	{
	    if (!have_fs(errp)) {
	        return;
	    }
	
	    if (NULL==target) {
	        error_setg(errp, QERR_INVALID_PARAMETER_VALUE, "target", "a path");
	        return;
	    }
	
	    fs_event_fn(target);
	}
