---
layout: post
title:  hive笔记
date:   2016-5-13 10:43:27
category: "Hadoop"
---

![](http://geleeq.github.io/blog/post_res/images/hadoop/hivearch.jpg)
![](http://geleeq.github.io/blog/post_res/images/hadoop/hivemetastore.jpg)

- **embedded metastore** 每次只有一个内嵌Derby数据库可以访问某个磁盘上的数据库文件
- **local metastore** metastore服务仍然和Hive服务运行在同一个进程中（javax.jdo.option.*配置指定数据库）
- **remote metastore** hive.metastore.uris配置远程metastore服务器（thrift://host:port）

### 与传统数据库相比

- schema on wirte

表的模式是在数据加载时强制确定，发现不符合模式时，拒绝加载数据；有利于提升查询性能，因为数据库可以对列进行索引。

- schema on read

Hive对数据的验证并不在加载数据时进行，而在查询时进行。因为查询尚未确定，因此不能决定使用何种索引。