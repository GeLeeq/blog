---
layout: post
title:  zookeeper笔记
date:   2016-5-16 09:43:27
category: "Hadoop"
---

1.配置（先在一台节点上配置）
	zookeeper的默认配置文件为zookeeper/conf/zoo_sample.cfg，需要将其修改为zoo.cfg。其中各配置项的含义，解释如下：

	tickTime：CS通信心跳时间
	Zookeeper 服务器之间或客户端与服务器之间维持心跳的时间间隔，也就是每个 tickTime 时间就会发送一个心跳。tickTime以毫秒为单位。
	tickTime=2000  
	
	initLimit：LF初始通信时限
	集群中的follower服务器(F)与leader服务器(L)之间初始连接时能容忍的最多心跳数（tickTime的数量）。
	initLimit=5  
	
	syncLimit：LF同步通信时限
	集群中的follower服务器与leader服务器之间请求和应答之间能容忍的最多心跳数（tickTime的数量）。
	syncLimit=2  
	 
	dataDir：数据文件目录
	Zookeeper保存数据的目录，默认情况下，Zookeeper将写数据的日志文件也保存在这个目录里。
	dataDir=/home/michael/opt/zookeeper/data  
	
	clientPort：客户端连接端口
	客户端连接 Zookeeper 服务器的端口，Zookeeper 会监听这个端口，接受客户端的访问请求。
	clientPort=2181 
	
	1.2修改配置文件（zoo.cfg）
		dataDir=/home/hadoop/app/zookeeper-3.4.5/data
		
		server.1=itcast05:2888:3888 //心跳端口、选举端口
		server.2=itcast06:2888:3888
		server.3=itcast07:2888:3888
	
	1.3在（dataDir=/home/hadoop/app/zookeeper-3.4.5/data）创建一个myid文件，
		里面内容是server.N中的N（server.2里面内容为2）
		echo "1" > myid
	
	1.4将配置好的zk拷贝到其他节点
		scp -r /itcast/zookeeper-3.4.5/ itcast06:/itcast/
		scp -r /itcast/zookeeper-3.4.5/ itcast07:/itcast/
	
	1.5注意：在其他节点上一定要修改myid的内容
		在itcast06应该讲myid的内容改为2 （echo "2" > myid）
		在itcast07应该讲myid的内容改为3 （echo "3" > myid）
		
2.启动集群  
	分别启动zk 	`bin/zkServer.sh start`  
	查看状态 `bin/zkServer.sh status`

**zookeeper的最主要功能：**  
1、保管客户端提交的数据（极其少量的数据）	每一份数据在zookeeper叫做一个znode
znode之间形成一种树状结构（类似于文件树）  
比如：  
/lock(znode名) host001(znode中存的数据)

/lock/last_acc(znode名)    host008(znode中存的数据)