---
layout: post
title: Python两个内置函数—locals和globals
category: python
tags: python
keywords: 
description: 
---

## 命名空间 ##

与其他语言一样，python有命名空间的概念。


方法内的变量，作用范围为整个方法，这个命名空间叫局部命名空间  
模块内的函数、类等，作用范围为整个模块，其所属的命名空间叫全局命名空间  
另外就是内置的命名空间，他的作用范围是所有的模块。

python解析的时候，会按照上述顺序依次查找  
当所以命名空间都查找不到时，抛出There is no variable named 'x' 

## 导入模块 ##

#### import module ####

原有命名空间保持不变，导入者变量引用需要加模块名

#### from module import ####

会将指定的方法和属性导入到当前命名空间中，故不需要加模块引用

## 应用举例 ##

![](http://i.imgur.com/uYL6NF8.png)  
如上图，通过globals()获取到全局的方法