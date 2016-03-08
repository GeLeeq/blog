---
layout: post
title:  Linux 常用命令
date:   2016-02-20 12:12:00
category: "Linux"
---
## linux命令 ##

- 系统信息查看:

	`uname -a` ： 查看系统内核版本号及系统名称

	`cat /proc/version`： 查看目录"/proc"下version的信息，也可以得到当前系统的内核版本号及系统名称

补充说明：

　　/proc文件系统，它不是普通的文件系统，而是系统内核的映像，也就是说，该目录中的文件是存放在系统内存之中的，它以文件系统的方式为访问系统内核数据的操作提供接口。而我们使用命令“uname -a"的信息就是从该文件获取的，当然用方法二的命令直接查看它的内容也可以达到同等效果.另外，加上参数"a"是获得详细信息，如果不加参数为查看系统名称。

- Linux删除所有文件及解压war包:

  	`rm -rf *`

  	`jar -xvf  abc.war`


- linux tar命令:


	      -c: 建立压缩档案
    	  -x：解压
    	  -t：查看内容
    	  -r：向压缩归档文件末尾追加文件
    	  -u：更新原压缩包中的文件
	
	  
	这五个是独立的命令，压缩解压都要用到其中一个，可以和别的命令连用但只能用其中一个。下面的参数是根据需要在压缩或解压档案时可选的。
	
      	-z：有gzip属性的
	  	-j：有bz2属性的
	  	-Z：有compress属性的
	  	-v：显示所有过程
	  	-O：将文件解开到标准输出
	
	下面的参数-f是必须的
	
	  	-f: 使用档案名字，切记，这个参数是最后一个参数，后面只能接档案名。

- mv [-r] 移动文件 [目录] 
- cp [-r] 复制文件 [目录]
   
5. linux文件目录访问权限：
   chown -R root .    修改成root用户        
   chown -R root:mysql /data  修改成mysql组的root用户
   chgrp -R mysql . 修改成mysql组
   
6. linux添加用户组和用户
   groupadd mysql
   useradd mysql -g mysql

7. chmod a+x file  使所有用户都有执行权限，会有安全问题。
   chmod o+x file  是拥有者有执行权限。
另外也可以使用sh file.sh命令执行文件，需要有该文件读权限。

8. 添加环境JAVA_HOME等环境变量
  在/etc/profile文件末尾加入：
  JAVA_HOME=/usr/local/jdk1.5.0_05
  PATH=$JAVA_HOME/bin:$PATH
  CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
  export JAVA_HOME
  export PATH
  export CLASSPATH
  
	保存退出; 执行以下命令:

			source /etc/profile

- 查看端口是否被占用
  netstat –apn | grep 8080

- 免密码ssh设置

	现在确认能否不输入口令就用ssh登录localhost:

    	$ ssh localhost

	如果不输入口令就无法用ssh登陆localhost，执行下面的命令：

		$ ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa 
		$ cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_keys

- 结合find删除大量文件
	

	1.find命令的一般形式

			find pathname -options [-print -exec -ok ...]

	2.find命令的参数
	
			pathname: find命令所查找的目录路径。例如用.来表示当前目录，用/来表示系统根目录。
			-print： find命令将匹配的文件输出到标准输出。
			-exec： find命令对匹配的文件执行该参数所给出的shell命令。相应命令的形式为'command' { } \;
					注意{   }和\；之间的空格。
			-ok： 和-exec的作用相同，只不过以一种更为安全的模式来执行该参数所给出的shell命令，
					在执行每一个命令之前，都会给出提示，让用户来确定是否执行。

	3.find命令选项
	
			-name 按照文件名查找文件。
			$ find ./ -name a.txt -exec rm -fv {} \;   //删除当前目录及其子目录下所有a.txt文件