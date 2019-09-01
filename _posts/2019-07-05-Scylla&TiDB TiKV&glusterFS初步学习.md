---
layout:     post
title:      "Scylla&TiDB TiKV&CQL 入门"
subtitle:   " 无比硬核的 TiDB 的官方文档与超强的技术 "
date:       2019-04-01 11:00:00
author:     "Ubik"
header-img: "img/home-bg-o.jpg"
catalog: true
comments: true
tags:
    - 实习
    - 数据库
    - 笔记
---

实习的第一件事就是读本组产品的 API 文档，然后学习系统用到的底层框架。这些东西之前从没看过
# TiDB
# 概念
- OLTP (Online Transactional Processing) 

- OLAP (Online Analytical Processing) 


通过单机的 RocksDB，我们可以将数据快速地存储在磁盘上；通过 Raft，我们可以将数据复制到多台机器上，以防单机失效。数据的写入是通过 Raft 这一层的接口写入，而不是直接写 RocksDB。通过实现 Raft，我们拥有了一个分布式的 KV，现在再也不用担心某台机器挂掉了。

对于一个 KV 系统，将数据分散在多台机器上有两种比较典型的方案：一种是按照 Key 做 Hash，根据 Hash 值选择对应的存储节点；另一种是分 Range，某一段连续的 Key 都保存在一个存储节点上。TiKV 选择了第二种方式，将整个 Key-Value 空间分成很多段，每一段是一系列连续的 Key，我们将每一段叫做一个 Region，并且我们会尽量保持每个 Region 中保存的数据不超过一定的大小(这个大小可以配置，目前默认是 64mb)。每一个 Region 都可以用 StartKey 到 EndKey 这样一个左闭右开区间来描述。

TiKV 集群是 TiDB 数据库的分布式 KV 存储引擎，数据以 Region 为单位进行复制和管理，每个 Region 会有多个 Replica（副本），这些 Replica 会分布在不同的 TiKV 节点上，其中 Leader 负责读/写，Follower 负责同步 Leader 发来的 raft log

![插图]({{site.baseurl}}/img/in-post/3DE33801F12030F0CB76AFE53384C6AA.jpg)

在 KV 结构上保存 Table 以及如何在 KV 结构上运行 SQL 语句

# glusterFS
# 简介
GlusterFS (Gluster File System) 是一个开源的分布式文件系统，是 Scale-Out 存储解决方案 Gluster 的核心，具有强大的横向扩展能力，通过扩展能够支持数PB存储容量和处理数千客户端。借助 TCP/IP 或 InfiniBand RDMA 网络将物理分布的存储资源聚集在一起，使用单一全局命名空间来管理数据。

# scylla介绍
开源的分布式NoSQL数据存储。与Apache Cassandra兼容，同时实现更高的吞吐量和更低的延迟。
[号称十倍性能于Cassandra的ScyllaDB，究竟祭出了哪些技术”利器”？](http://www.nosqlnotes.com/technotes/scylla-highlights/)
# Cassandra
Apache Cassandra（社区内一般简称为C*）是一套开源分布式NoSQL数据库系统。它最初由Facebook开发，用于储存收件箱等简单格式数据，具有高度可扩展性，可用于管理大量的结构化数据。它提供高可用性，没有单点故障,集Google BigTable的数据模型与Amazon Dynamo的完全分布式架构于一身。
Cassandra使用了Google 设计的 BigTable的数据模型，与面向行(row)的传统的关系型数据库或键值存储的key-value数据库不同，Cassandra使用的是宽列存储模型(Wide Column Stores)[8]，每行数据由row key唯一标识之后，可以有最多20亿个列[11]，每个列有一个column key标识，每个column key下对应若干value。这种模型可以理解为是一个二维的key-value存储，即整个数据模型被定义成一个类似map<key1, map<key2,value>>的类型。 
[wiki百科](https://zh.wikipedia.org/wiki/Cassandra)
# Cassandra Query Language (CQL) 语法
与SQL语法类似，但不支持join和子查询
结构类似HBase，row x key x value
常用命令
- describe tables
- describe keyspaces
- source 执行包含命令的文件

键空间是一个定义节点上数据复制的命名空间，集群中每个节点对应一个键空间。
键、表、行关系通过expand on expand off 可以看的很明白
如下
```sql
cqlsh:tutorialspoint> SELECT * from emp
                  ... ;

@ Row 1
-----------+------------
 emp_id    | 4
 emp_city  | Pune
 emp_name  | rajeev
 emp_phone | 9848022331
 emp_sal   | 30000

@ Row 2
-----------+------------
 emp_id    | 3
 emp_city  | null
 emp_name  | null
 emp_phone | null
 emp_sal   | 50000

(2 rows)
cqlsh:tutorialspoint> SELECT * from emp  ;

 emp_id | emp_city | emp_name | emp_phone  | emp_sal
--------+----------+----------+------------+---------
      4 |     Pune |   rajeev | 9848022331 |   30000
      3 |     null |     null |       null |   50000

```
# 创建keyspace
```sql
CREATE KEYSPACE “KeySpace Name”
WITH replication = {'class': ‘Strategy name’, 'replication_factor' : ‘No.Of   replicas’};

CREATE KEYSPACE “KeySpace Name”
WITH replication = {'class': ‘Strategy name’, 'replication_factor' : ‘No.Of  replicas’}

AND durable_writes = ‘Boolean value’;
```
其中复制选项用于指定副本位置策略和所需副本的数量，另一个是是否持久化
# 修改keyspace
```sql
ALTER KEYSPACE “KeySpace Name”
WITH replication = {'class': ‘Strategy name’, 'replication_factor' : ‘No.Of  replicas’};
```
对创建时的两个属性，复制和持久化作出修改
# 删除keysopce
```sql
DROP KEYSPACE “KeySpace name”
```

## 创建表
语法与实例如下
```sql
CREATE (TABLE | COLUMNFAMILY) <tablename>
('<column-definition>' , '<column-definition>')
(WITH <option> AND <option>)


CREATE TABLE emp(
   emp_id int PRIMARY KEY,
   emp_name text,
   emp_city text,
   emp_sal varint,
   emp_phone varint
   );
```
## 修改表
语法与实例如下
```sql
ALTER (TABLE | COLUMNFAMILY) <tablename> <instruction>

alter table tablename add new column datatype

alter table emp add emp_email text

alter table tablename drop column name

alter table emp drop emp_email

```
## 删除表
语法与实例如下
```sql
drop table tablename
```
## 截断表
(把数据都弄没）
```sql
truncate tabblename
```

## 创建、删除索引
```sql
create index identifier on tablename
create index name on emp1
drop index identifier
drop index name
```
## 批处理
```sql
begin batch 
<insert-stmt>/ <update-stmt>/ <delete-stmt>
apply batch
```
# CURD 
```sql
insert into tablename (column1name,column2name...) values (value1,value2...) using <option>
INSERT INTO emp (emp_id, emp_city , emp_name , emp_phone, emp_sal ) values(250,'nanjing','nidie',1234567,88888)  ;(cql中使用单引号）

update tablename set columnname=value... where condition
update emp set emp_city = 'PPPPP' WHERE emp_id =250 ;

select x from tablename where condition
WHERE子句只能用于作为主键的一部分或在其上具有辅助索引的列。
不给where子句添加索引的情况下，会报错如下，加索引后就好了。
```
> InvalidRequest: Error from server: code=2200 [Invalid query] message="Cannot execute this query as it might involve data filtering and thus may have unpredictable performance. If you want to execute this query despite the performance unpredictability, use ALLOW FILTERING"

```sql
删除数据
delete x from identifier where condition
与更新一样，不加where会报语法错误，有where无条件变量会报no viable alternative at input ';'
```

# 数据类型
CQL提供了许多内置类型包括集合，也可自定义，命令如下
    CREATE TYPE -创建用户定义的数据类型。

    ALTER TYPE -修改用户定义的数据类型。

    DROP TYPE -删除用户定义的数据类型。

    DESCRIBE TYPE -描述用户定义的数据类型。

    DESCRIBE TYPES -描述用户定义的数据类型。

## list 
适用于保持元素顺序且值被多次存储，格式同内建类型，只是插入时用[]括起来，中间,分割。更新时如下

```sql
UPDATE data
... SET email = email +['xyz@tutorialspoint.com']
... where name = 'ramu';
```

## set 
根据常识，无序且不重复
```sql
CREATE TABLE data2 (name text PRIMARY KEY, phone set<varint>);
花括号{}分割
INSERT INTO data2(name, phone)VALUES ('rahman',    {9848022338,9848022339});
更新时同list
tutorialspoint> UPDATE data2
   ... SET phone = phone + {9848022330}
   ... where name = 'rahman';

```

## map
KV对
```sql
CREATE TABLE data3 (name text PRIMARY KEY, address
map<timestamp, text>);
INSERT INTO data3 (name, address)
   VALUES ('robin', {'home' : 'hyderabad' , 'office' : 'Delhi' } );
   UPDATE data3
   ... SET address = address+{'office':'mumbai'}
   ... WHERE name = 'robin';
```
# 自定义数据类型
```sql
CREATE TYPE <keyspace name>. <data typename>
( variable1, variable2)

CREATE TYPE tutorialspoint.card_details (
   num int,
   pin int,
   name text,
   cvv int,
   phone set<int>
   );
   
ALTER TYPE typename
ADD field_name field_type; 

DESCRIBE_TYPES命令验证所有用户定义的数据类型的列表

```














# Seastar
一个用于在现代多核机器上，编写高效复杂的服务器应用程序的C++库。它允许构建高度复杂的服务端应用程序，同时性能很好。它是事件驱动的，允许以相对直接的方式编写非阻塞的异步代码。
Scylladb第一个使用了Seastar，它是对Apache Cassandra的重写。 Cassandra是一个非常复杂的应用程序，然而，通过Seastar，我们能够让吞吐量提高10倍，同时显著的降低一致性的延迟。
> 异步服务器应用程序的作者面临着今天仍面临的两大挑战：
>
>  复杂性：编写简单的异步服务器非常简单。但是编写一个复杂的异步服务器是非常困难的。单个连接的处理，而不是一个简单易读的函数调用，现在涉及大量的小型回调函数和一个复杂的状态机，以记住每个事件发生时需要调用哪个函数。
>
>  非阻塞：每个内核只有一个线程，对于服务器应用程序性能很重要，因为上下文切换很慢。但是，如果我们每个内核只有一个线程，则事件处理函数不能阻塞，否则内核将保持空闲状态。但是一些现有的编程语言和框架让服务器作者别无选择，只能使用阻塞函数，因此也不能使用多线程。例如，Cassandra被编写为一个异步服务器应用程序;但是因为磁盘I/O是用mmaped文件实现的，所以在访问时可以不受控制地阻塞整个线程，它们被迫在每个CPU上运行多个线程。