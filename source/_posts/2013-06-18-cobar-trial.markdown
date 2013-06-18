---
layout: post
title: "cobar试用记录"
date: 2013-06-18 21:10
comments: true
categories: tdal
---
# 1.介绍 #

Cobar是关系型数据的分布式处理系统，它可以在分布式的环境下看上去像传统数据库一样为您提供海量数据服务。


- 产品在阿里巴巴B2B公司已经稳定运行了3年以上。


- 目前已经接管了3000+个MySQL数据库的schema，为应用提供数据服务。


- 据最近统计cobar集群目前平均每天处理近50亿次的SQL执行请求。

详细的产品文档见产品主页 [http://code.alibabatech.com/wiki/display/cobar/Home](http://code.alibabatech.com/wiki/display/cobar/Home "http://code.alibabatech.com/wiki/display/cobar/Home")

# 2.产品约束 #
- 不支持跨库情况下的join、分页、排序、子查询操作。
- SET语句执行会被忽略，事务和字符集设置除外。
- 分库情况下，insert语句必须包含拆分字段列名。
- 分库情况下，update语句不能更新拆分字段的值。
- 不支持SAVEPOINT操作。
- 暂时只支持MySQL数据节点。
- 使用JDBC时，不支持rewriteBatchedStatements=true参数设置(默认为false)。
- 使用JDBC时，不支持useServerPrepStmts=true参数设置(默认为false)。
- 使用JDBC时，BLOB, BINARY, VARBINARY字段不能使用setBlob()或setBinaryStream()方法设置参数。
# 3.功能概述 #
Cobar是关系型数据的分布式处理系统，它可以在分布式的环境下看上去像传统数据库一样为您提供海量数据服务。
![](http://code.alibabatech.com/wiki/download/attachments/7671478/deploy.jpg?version=1&modificationDate=1341458291000)
## 3.1 Cobar解决的问题 ##
- 分布式：Cobar的分布式主要是通过将表放入不同的库来实现：

1.Cobar支持将一张表水平拆分成多份分别放入不同的库来实现表的水平拆分

2.Cobar也支持将不同的表放入不同的库

3.多数情况下，用户会将以上两种方式混合使用
这里需要强调的是，Cobar不支持将一张表，例如test表拆分成test_1, test_2, test_3.....放在同一个库中，必须将拆分后的表分别放入不同的库来实现分布式。


- HA：
在用户配置了MySQL心跳的情况下，Cobar可以自动向后端连接的MySQL发送心跳，判断MySQL运行状况，一旦运行出现异常，Cobar可以自动切换到备机工作。但需要强调的是：

1.Cobar的主备切换有两种触发方式，一种是用户手动触发，一种是Cobar的心跳语句检测到异常后自动触发。那么，当心跳检测到主机异常，切换到备机，如果主机恢复了，需要用户手动切回主机工作，Cobar不会在主机恢复时自动切换回主机，除非备机的心跳也返回异常。

2.Cobar只检查MySQL主备异常，不关心主备之间的数据同步，因此用户需要在使用Cobar之前在MySQL主备上配置双向同步，详情可以参阅MySQL参考手册。

## 3.2 Cobar逻辑层次图 ##
![](http://code.alibabatech.com/wiki/download/attachments/7671478/schema.jpg?version=2&modificationDate=1341912304000)

- dataSource：数据源，表示一个具体的数据库连接，与一个物理存在的schema一一对应。
- dataNode：数据节点，由主、备数据源，数据源的HA以及连接池共同组成，可以将一个
- dataNode理解为一个分库。
- table：表，包括拆分表(如tb1，tb2)和非拆分表。
- tableRule：路由规则，用于判断SQL语句被路由到具体哪些datanode执行。
- schema：cobar可以定义包含拆分表的schema(如schema1)，也可以定义无拆分表的schema(如schema2)。
- 以上层次关系具有较强的灵活性，用户可以将表自由放置不同的datanode，也可将不同的datasource放置在同一MySQL实例上。

## 2.3 Cobar使用注意事项 ##

1. 请注意表的拆分方式，一张表水平拆分多份到不同的库中，而不是放入同一个库中。
2. 如果使用HA功能，请在MySQL主备之间配置双向同步

# 4.Cobar试用记录 #

## 4.1 解决单表数据量过大问题 ##

1. 场景说明.

在即使通讯系统中，需要存储历史会话记录，但是历史消息会话的数据量会随时间和用户增长非常快，如果活跃用户数据量较大，而且用户使用即使通讯较为频繁的话，则数据增长较快，假设即时通讯系统中活跃用户量是30万，每位用户每天产生的会话消息数量是100条记录，则改表每日增长的数据量是30万*100=3000万条数据，按照这种增长速度，则历史消息表的增长会很快超过单表数据量限制，因此必须通过分表实现将数据分布到不同的数据中。

2. 实现策略.

假设要分的表名是message,表结构如下所示：

field Type Comment

userId bigint(20) NOT NULL

from varchar(100) NOT NULL

to varchar(100) NOT NULL

msg varchar(500) NOT NULL

createTime datetime NOT NULL

SQL语句脚本如下：

    CREATE TABLE `message` (
      `userId` bigint(20) NOT NULL,
      `from` varchar(100) NOT NULL,
      `to` varchar(100) NOT NULL,
      `msg` varchar(500) NOT NULL,
      `createTime` datetime NOT NULL,
      KEY `index_userId` (`userId`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8



将表message中的数据分布到3个数据库中。

3. 