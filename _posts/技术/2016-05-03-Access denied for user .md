---
layout: post
title: ERROR 1045(28000)-数据库连不上
category: 技术
tags: openstack
keywords: Access denied for user 'root@localhost' (using password no )
description: 
---

这两天按照官网安装openstack遇到个数据库问题，断断续续折腾了好几天，今天终于解决了，记录下来，给有缘人。

安装完MariaDB后，输入mysql -u -r   ****后，会报出上述错误

查阅很多文档，有说需要更新密码的，又说需要重新授权的，按照操作无效

但是

    /etc/init.d/mysql stop   (service mysqld stop )
    /usr/bin/mysqld_safe --skip-grant-tables

可以登录，然后确认是哪里权限有问题，无奈之前SQL只是用，没有管理员层面的经验

结果，按照如下方法修改,问题规避

在my.cnf文件中增加修改如下字段
    
    [mysqladmin]
     user = root
     password = mysqlrootpassword
    [mysql]
     user = root
     password = mysqlrootpassword
    [mysqldump]
     user = root
     password = mysqlrootpassword
    





