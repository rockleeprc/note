
## cluster

### mint cluster
	centos0 172.16.1.10		保留
	centos1 172.16.1.11		jdk1.8 	zookeeper		redis(master:6379/slave:6380)
	centos2 172.16.1.12 	jdk1.8 	zookeeper		
	centos3 172.16.1.13		jdk1.8 	zookeeper		
	centos4 172.16.1.14		jdk1.8 	mysql
	centos5 172.16.1.15		jdk1.8 	
	centos6 172.16.1.16		jdk1.8 	

### hadoop cluster
	hdp01	192.168.33.11	zookeeper	redis(master)
	hdp02	192.168.33.12	zookeeper	redis(slave)
	hdp03	192.168.33.13	zookeeper	

## 网络配置
* 主机所在网段: 172.16.1.0/24
* nat网段: 10.0.2.0/24

## 主机克隆
	# 删除克隆主机原有的网卡信息（一般为eth0、eth1），保留新生成的网卡信息
	/etc/udev/rules.d/70-persistent-net.rules
	# 修改NAME名称为源网卡名称（一般为eth0、eth1）
	SUBSYSTEM=="net", ACTION=="add", DRIVERS=="?*", ATTR{address}=="08:00:27:77:89:20", ATTR{type}=="1", KERNEL=="eth*", NAME="eth0"

	# 修改eth0网卡的mac地址为/etc/udev/rules.d/70-persistent-net.rules中所对应的ATTR{address}
	/etc/sysconfig/network-scripts/ifcfg-eth0
	HWADDR=08:00:27:77:89:20

	# 重启网卡服务
	source 修改的配置文件
	service network restart
	/etc/init.d/network restart

## 修改主机名
* /etc/sysconfig/network
* /etc/hosts

## ssh登录
	ssh 172.16.1.11 -l root
	ssh -l root 192.168.2.216 pwd

## ssh互信
	# 生成ssh公私钥
	[root@centos1 ~]# ssh-keygen -t rsa
	# 将公钥复制到本地authorized_keys文件
	[root@centos1 ~]# ssh-copy-id -i .ssh/id_rsa.pub root@172.16.1.11
	# 将每台主机的公钥返送给centos1,cengos1就拥有集群中所有主机的公钥
	[root@centos2 ~]# ssh-copy-id -i .ssh/id_rsa.pub root@172.16.1.11
	# centos1的uthorized_keys发送给其它机器
	[root@centos1 .ssh]# scp authorized_keys root@centos2:~/.ssh/

## JDK
	# 系统自带的OpenJDK
	[root@centos1 local]# java -version
	java version "1.7.0_99"
	OpenJDK Runtime Environment (rhel-2.6.5.1.el6-x86_64 u99-b00)
	OpenJDK 64-Bit Server VM (build 24.95-b01, mixed mode)

	# rpm:管理套件    
	# -qa:使用询问模式，查询所有套件
	#	grep:查找文件里符合条件的字符串
	# java:查找包含java字符串的文件
	[root@centos1 local]# rpm -qa | grep java
	tzdata-java-2016c-1.el6.noarch
	java-1.7.0-openjdk-1.7.0.99-2.6.5.1.el6.x86_64
	java-1.6.0-openjdk-1.6.0.38-1.13.10.4.el6.x86_64

	## 删除所有openJDK相关的套件
	# rpm:管理套件  
	# -e:删除指定的套件
	#	--nodeps:不验证套件档的相互关联性
	[root@centos1 local]# rpm -e --nodeps java-1.7.0-openjdk-1.7.0.111-2.6.7.8.el7.x86_64
	# 一条命令全部删除
	[root@centos1 local]# #rpm -e --nodeps `rpm -qa | grep java`
	# 没有任何jdk
	[root@centos1 local]# java -version
	-bash: /usr/bin/java: 没有那个文件或目录

	# 修改解压后jdk文件夹组权限
	[root@centos1 local]# chown -R root:root jdk1.8.0_131/

	# 配置环境变量
	JAVA_HOME=/usr/local/jdk1.8.0_131
	JRE_HOME=/usr/local/jdk1.8.0_131/jre
	CLASSPATH=.
	PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
	export JAVA_HOME
	export JRE_HOME
	export CLASSPATH
	export PATH

	# 将centos1配置好的jdk发送给其它机器
	[root@centos1 local]# scp -r /usr/local/jdk1.8.0_131 root@centos2:/usr/local
