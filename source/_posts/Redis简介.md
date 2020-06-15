---
title: Redis简介
date: 2020-06-14 15:38:18
tags: redis
cover: 
---

##### C语言编写，完全开源免费，遵循BSD协议（伯克利软件发行版），高性能的NoSQL(非关系型数据库) key-value数 
##### 1.0 出现原因分析：
---

 - High performance 
<br>对数据库高并发读写的需求
 - Hug storage <br>对海量数据的高效率存储和访问的需求

 - High Scalabilty && High Availability <br>对数据库的高可扩展性和高可用性的需求
##### 2.0 NoSQL类别
---

| 类别                 | 优点                                         | 缺点                                                     | 典型应用                                     | 数据模型                           | 相关产品 |
| -------------------- | -------------------------------------------- | -------------------------------------------------------- | -------------------------------------------- | ---------------------------------- | -------- |
| key-value 存储数据库 | 快速查询                                     | 存储的数据缺少结构化                                     | 内容缓存，主要用于处理大量数据的高访问负载   | 一系列的键值对                     | Redis    |
| 列存储数据库         | 查找速度快，可扩展性强，更容易进行分布式扩展 | 功能相对局限                                             | 分布式的文件系统                             | 以列簇式存储，将同一列数据存在一起 | hbase    |
| 文档型数据库         | 数据结构要求不严格                           | 查询性能不高，并缺乏统一的查询语法                       | web 应用（与key-value类似，value是结构化的） | 一系列键值对                       | MongoDB  |
| 图形数据库           | 利用图结构相关算法                           | 需要对整个图做计算才能得出结果，不容易做分布式的集群方案 | 社交网络                                     | 图结构                             | Neo4J    |
##### 3.0 Redis特点
###### 优点
- 高性能
<br>官方测试11W次/s读操作，8w次/s写操作
 - 数据类型丰富 <br>支持String,Set,Hash,List及Ordered Set 类型操作

 - 丰富特性 <br>支持publish/subscribe,通知，key过期等
 - 高速读写  <br>Redis使用自己实现的分离器，代码量短，没有使用lock效率高
 - 原子性 <br>所有操作都是原子性的，单个操作是原子性的；多个操作通过MULTI和EXEC实现事务原子性操作。
###### 缺点
 - 持久化 <br>Redis直接将数据存储到内存中，要将数据保存到磁盘上有两种持久化方式。RDB/AOF,后期有专题讲解。
 - 内存占用过高 
##### 3.0 Redis安装
---
###### 3.0.1 采用C语言开发，Linux系统下需要编译，需要有GCC环境

```
 $ yum -y install gcc automake autoconf libtool make
```
###### 注意：
</br>运行yum时出现/var/run/yum.pid已被锁定，pid为XXXX正在运行的问题解决
```
$ rm -f /var/run/yum.pid
```
###### 3.0.2 安装
```
$ wget http://download.redis.io/releases/redis-5.0.7.tar.gz
$ tar xzf redis-6.0.5.tar.gz -C 指定解压目录
$ cd redis-5.0.7
$ make
```
###### 3.0.3 单机服务启动
```
$ src/redis-server
```
###### 3.0.4 客户端启动及简单功能使用
###### 方式一
---
```
$ src/redis-cli
redis> set foo bar
OK
redis> get foo
"bar"
```
###### 方式二：客户端命令语法
---
```
$ src/redis-cli -h ip地址 -p 端口号 -a 密码（设置的话）
redis> set foo bar
OK
redis> get foo
"bar"
```
##### 4.0 Redis内存维护策略
---
###### 4.0.1 设置数据超时时间
```
--常用方式
  expire key time (s为单位)
--字符串
  setex(String key，int time ,String value )
```
- 除字符串外，都需要以expire来设置过期时间
- 未设置则默认为永不过期
- 设置后取消过期时间设置：persist key
###### 4.0.2 LRU算法动态删除数据
---
> 内存管理的一种页面置换算法，存在内存中但不使用的数据（内存块）叫做LRU，操作系统会移出LRU数据来加载其它数据 

- volatile-lru: 设定超时时间的数据中，删除最不常使用的数据
- allkeys-lru:  查询所有的key中最近最不常使用的数据删除（使用广泛）