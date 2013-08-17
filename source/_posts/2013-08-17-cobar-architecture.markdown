---
layout: post
title: "cobar架构设计"
date: 2013-08-17 08:07
comments: true
categories: ddal
---
# 前言 #

最近打算学习一下cobar的源代码，方便以后扩展或者改造cobar以符合我们的需求，增加一些目前cobar不支持的特性，比如读写分离等特性。

以前不知道该如何下手看一个大型开源项目的源代码，经过一段时间的摸索之后，我认为要学习一个开源项目的代码，直接下手看代码会很盲目，不容易看懂。按照以下方法也许更好。

1.先学习开源项目的原理或者协议。了解开源项目的大概实现原理，以及它遵循的协议，比如我原来阅读过的CAS，先了解CAS的协议及原理，再去看源代码就容易看懂了；再比如我之前阅读过的openfire源码，在看之前我们先学习XMPP协议的原理，再去看源代码的时候，就更容易看懂代码。

2.学习开源项目的架构设计。先通过官方或者别人分享的开源项目的架构设计，有助于从宏观上了解整个项目源代码的结构，这样在看源代码的时候就不容易迷失方向，也更容易读懂代码。很幸运的是cobar这个项目的文档还是相对比较完善的。本文也是在学习了cobar的架构设计之后整理出来的，大部分内容和图都是来源于cobar的文档，加上自己的一些理解。

3.带有目的性的阅读源代码。因为一个大型开源项目的源代码都比较多，很难短时间内全部阅读完，带有目的性的去阅读更容易获得成就感，比如你的目的是解决一个技术问题，扩展一个不支持的特性等等。

# cobar架构 #
## 系统模块架构图 ##

![](http://code.alibabatech.com/wiki/download/attachments/7671891/WorkFlow.JPG?version=2&modificationDate=1340102728000)

Front-end Communication: Cobar与前端应用的通信模块

SQL Parser : SQL解析，提取SQL中的重要信息

SQL Router : 决定SQL语句被分发到哪些库执行

SQL Excutor : 执行SQL语句

Result-Merger : 合并多个库的执行结果

DataNode，HA Pool，JDBC : 后端数据访问

其实让我们自己来设计一个这样的数据库代理服务器的话估计也会类似于这样设计，因为它确实需要这些功能模块组成。


### 前端通信 ###
负责从客户端读入SQL语句，并将执行结果集发送给客户端，遵循MySQL协议
![](http://code.alibabatech.com/wiki/download/attachments/7671891/frontend2.JPG?version=1&modificationDate=1340093766000)

### 解析、路由 ###

解析SQL即提取SQL信息

![](http://code.alibabatech.com/wiki/download/attachments/7671891/parser-extract.jpg?version=1&modificationDate=1340093766000)

路由并重组SQL，减少不必要的查询

![](http://code.alibabatech.com/wiki/download/attachments/7671891/paser-reconstruct.JPG?version=2&modificationDate=1340096890000)

### 后端数据访问 ###
后端连接池，连接复用，提升效率

在datanode内部做HA，细粒度的FailOver机制，一旦主库有异常，立即切换到备库

用户可自由将不同datanode的主、备库放在同一数据库实例，分散MySQL实例上的压力
![](http://code.alibabatech.com/wiki/download/attachments/7671891/backend.JPG?version=4&modificationDate=1340103363000)

## 系统实现 ##

### 各个模块数据流图 ###
![](http://code.alibabatech.com/wiki/download/attachments/7671478/data+flow.jpg?version=1&modificationDate=1342764051000)

### 网络通讯模块 ###
![](http://code.alibabatech.com/wiki/download/attachments/7671478/thread+flow.jpg?version=3&modificationDate=1342754569000)

### 路由与数据分布模块 ###
![](http://code.alibabatech.com/wiki/download/attachments/7671478/router.jpg?version=1&modificationDate=1342754871000)


### 结果合并与返回模块 ###
![](http://code.alibabatech.com/wiki/download/attachments/7671478/merge.JPG?version=2&modificationDate=1342098245000)