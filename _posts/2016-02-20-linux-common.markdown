---
layout: post
title:  Linux ��������
date:   2016-02-19 16:12:00
category: "Linux/CentOS"
---

1. ϵͳ��Ϣ�鿴
��� uname -a
���ã� �鿴ϵͳ�ں˰汾�ż�ϵͳ����

��� cat /proc/version
���ã� �鿴Ŀ¼"/proc"��version����Ϣ��Ҳ���Եõ���ǰϵͳ���ں˰汾�ż�ϵͳ����

����˵����
������proc�ļ�ϵͳ����������ͨ���ļ�ϵͳ������ϵͳ�ں˵�ӳ��Ҳ����˵����Ŀ¼�е��ļ��Ǵ����ϵͳ�ڴ�֮�еģ������ļ�ϵͳ�ķ�ʽΪ����ϵͳ�ں����ݵĲ����ṩ�ӿڡ�������ʹ�����uname -a"����Ϣ���ǴӸ��ļ���ȡ�ģ���Ȼ�÷�����������ֱ�Ӳ鿴��������Ҳ���Դﵽͬ��Ч��.���⣬���ϲ���"a"�ǻ����ϸ��Ϣ��������Ӳ���Ϊ�鿴ϵͳ���ơ�

2. Linux, ɾ�������ļ�����ѹwar��
 rm -rf *
  jar -xvf  abc.war


4.linux tar���
 
  -c: ����ѹ������
  -x����ѹ
  -t���鿴����
  -r����ѹ���鵵�ļ�ĩβ׷���ļ�
  -u������ԭѹ�����е��ļ�

  ������Ƕ��������ѹ����ѹ��Ҫ�õ�����һ�������Ժͱ���������õ�ֻ��������һ��������Ĳ����Ǹ�����Ҫ��ѹ�����ѹ����ʱ��ѡ�ġ�

  -z����gzip���Ե�
  -j����bz2���Ե�
  -Z����compress���Ե�
  -v����ʾ���й���
  -O�����ļ��⿪����׼���

  ����Ĳ���-f�Ǳ����

  -f: ʹ�õ������֣��мǣ�������������һ������������ֻ�ܽӵ�������

5. mv [-r] �ƶ��ļ�[Ŀ¼]
   cp [-r] �����ļ�[Ŀ¼]
   
6. linux�ļ�Ŀ¼����Ȩ�ޣ�
   chown -R root .    �޸ĳ�root�û�        
   chown -R root:mysql /data  �޸ĳ�mysql���root�û�
   chgrp -R mysql . �޸ĳ�mysql��
   
7. linux����û�����û�
   groupadd mysql
   useradd mysql -g mysql

8. chmod a+x file  ʹ�����û�����ִ��Ȩ�ޣ����а�ȫ���⡣
   chmod o+x file  ��ӵ������ִ��Ȩ�ޡ�
����Ҳ����ʹ��sh file.sh����ִ���ļ�����Ҫ�и��ļ���Ȩ�ޡ�

9.��ӻ���JAVA_HOME�Ȼ�������
  ��/etc/profile�ļ�ĩβ���룺
  JAVA_HOME=/usr/local/jdk1.5.0_05
  PATH=$JAVA_HOME/bin:$PATH
  CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
  export JAVA_HOME
  export PATH
  export CLASSPATH
 
  �����˳�; ִ����������:
  source /etc/profile

10. �鿴�˿��Ƿ�ռ��
  netstat �Capn | grep 8080

11. ������ssh����
����ȷ���ܷ�����������ssh��¼localhost:
$ ssh localhost
��������������޷���ssh��½localhost��ִ����������
$ ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa 
$ cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_keys

12. ���findɾ�������ļ�
1��find�����һ����ʽΪ��
find pathname -options [-print -exec -ok ...]

2��find����Ĳ�����
pathname: find���������ҵ�Ŀ¼·����������.����ʾ��ǰĿ¼����/����ʾϵͳ��Ŀ¼��
-print�� find���ƥ����ļ��������׼�����
-exec�� find�����ƥ����ļ�ִ�иò�����������shell�����Ӧ�������ʽΪ'command' { } \;��ע��{   }��\��֮��Ŀո�
-ok�� ��-exec��������ͬ��ֻ������һ�ָ�Ϊ��ȫ��ģʽ��ִ�иò�����������shell�����ִ��ÿһ������֮ǰ�����������ʾ�����û���ȷ���Ƿ�ִ�С�

3��find����ѡ��
-name �����ļ��������ļ���
$ find ./ -name a.txt -exec rm -fv {} \;   //ɾ����ǰĿ¼������Ŀ¼������a.txt�ļ�