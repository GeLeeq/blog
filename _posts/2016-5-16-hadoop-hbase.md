---
layout: post
title:  hbase笔记
date:  2016-5-16 16:54:22
category: "Hadoop"
---

1.配置hbase集群，要修改3个文件（首先zk集群已经安装好了）  
	注意：要把hadoop的hdfs-site.xml和core-site.xml 放到hbase/conf下
	
	1.1修改hbase-env.sh
	export JAVA_HOME=/usr/java/jdk1.7.0_55
	//告诉hbase使用外部的zk
	export HBASE_MANAGES_ZK=false
	
	vim hbase-site.xml
	<configuration>
		<!-- 指定hbase在HDFS上存储的路径 -->
        <property>
                <name>hbase.rootdir</name>
                <value>hdfs://ns1/hbase</value>
        </property>
		<!-- 指定hbase是分布式的 -->
        <property>
                <name>hbase.cluster.distributed</name>
                <value>true</value>
        </property>
		<!-- 指定zk的地址，多个用“,”分割 -->
        <property>
                <name>hbase.zookeeper.quorum</name>
                <value>weekend05:2181,weekend06:2181,weekend07:2181</value>
        </property>
	</configuration>
	
	vim regionservers    //regionserver主要用于读写表数据，所以与datanode放在相同机器
	weekend05
	weekend06
	weekend07
	
	1.2拷贝hbase到其他节点
		scp -r /weekend/hbase-0.96.2-hadoop2/ weekend02:/weekend/
		scp -r /weekend/hbase-0.96.2-hadoop2/ weekend05:/weekend/
		scp -r /weekend/hbase-0.96.2-hadoop2/ weekend06:/weekend/
		scp -r /weekend/hbase-0.96.2-hadoop2/ weekend07:/weekend/

2.将配置好的HBase拷贝到每一个节点并同步时间。

3.启动所有的hbase

	分别启动zk  
		./zkServer.sh start
	启动hdfs集群
		start-dfs.sh
	启动hbase，在主节点上运行：
		start-hbase.sh  //hbase-daemon.sh start master和hbase-daemon.sh start regionserver

4.通过浏览器访问hbase管理页面  
	`192.168.1.201:60010`

5.为保证集群的可靠性，要启动多个HMaster  (weekend02)  
	`hbase-daemon.sh start master`

6.启动shell客户端  
	`bin/hbase shell`