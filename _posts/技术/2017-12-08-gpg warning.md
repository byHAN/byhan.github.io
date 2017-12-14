---
layout: post
title: gpg WARNING
category: 技术
tags: 
keywords: 
description: 
---

编译内核的时候，遇到如下告警  

    gpg: WARNING: some OpenPGP programs can't handle a DSA key with this digest size
    
    ###
    ### Now generating a PGP key pair to be used for signing modules.
    ###
    ### If this takes a long time, you might wish to run rngd in the background to
    ### keep the supply of entropy topped up.  It needs to be run as root, and
    ### should use a hardware random number generator if one is available, eg:
    ###
    ### rngd -r /dev/hwrandom
    ###
    ### If one isn't available, the pseudo-random number generator can be used:
    ###
    ### rngd -r /dev/urandom
    ###
    
编译会卡住  
解决方法：新起xshell执行如下命令，可能需要安装rngd  

    rngd -r /dev/urandom


