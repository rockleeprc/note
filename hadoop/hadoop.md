## Linux准备

### 安装JDK

* 略

### 创建hadoop用户/组/分配root权限

* 添加hadoop组

  ```shell
  groupadd hadoop
  ```

* 添加hadoop用户

  ```shell
  groupadd hadoop
  ```

* 将hadoop用户添加到hadoop组、

  ```shell
  useradd hadoop -g hadoop
  ```

* 赋予hadoop用户root权限，修改/etc/sudoers 

  ```shell
  ## Allow root to run any commands anywhere
  root    ALL=(ALL)     ALL
  hadoop   ALL=(ALL)     ALL
  ```

### 关闭防火墙

* 查看开放的端口

  ```shell
  firewall-cmd--list-ports
  ```

* 查看防火墙状态

  ```shell
  firewall-cmd --state
  ```

* 关闭防火墙

  ```shell
  systemctl stop firewalld.service 
  ```

* 禁止firewallk开机启动

  ```shell
  systemctl disable firewalld.service
  ```

### 修改主机名

```shell
/etc/hostname
```

### 免密登录

```shell
# 生成ssh公私钥
ssh-keygen –t rsa
# copy公钥到授权服务器，同时也要对自己授权
ssh-copy-id node1
# 验证当前机器是否可以免密登录
ssh localhost
```

### ip到主机名称映射

```shell
/etc/hosts
192.168	.56.11 node1
192.168.56.12 node2
192.168.56.13 node3
# 将修改hosts拷贝到其它节点
scp  –r /etc/hosts  hadoop@node2：/etc/
```

## Zookeeper安装

* 修改配置

  ```shell
  dataDir=/data/zkData
  ## zk cluster ##
  server.1=node1:2888:3888
  server.2=node3:2888:3888
  server.3=node3:2888:3888
  ```

* 在dataDir目录创建myid文件，设置zNode id

* 分别在每个zNode节点启动

  ```shell
  zkServer.sh status
  ZooKeeper JMX enabled by default
  Using config: /usr/local/zookeeper/bin/../conf/zoo.cfg
  Mode: leader
  ```

## Hadoop安装

### CDH安装

#### 下载cloudera-manager

* 下载与linux版本对应的，比如centos7使用[cloudera-manager-centos7-cm5.16.1_x86_64.tar.gz](http://archive.cloudera.com/cm5/cm/5/cloudera-manager-centos7-cm5.16.1_x86_64.tar.gz)

```html
http://archive.cloudera.com/cm5/cm/5/
```

#### 下载CDH安装包

* 下载与linux版本对应的，比如centos7使用

  * [CDH-5.16.1-1.cdh5.16.1.p0.3-el7.parcel](http://archive.cloudera.com/cdh5/parcels/5.16.1/CDH-5.16.1-1.cdh5.16.1.p0.3-el7.parcel)
  * [CDH-5.16.1-1.cdh5.16.1.p0.3-el7.parcel.sha1](http://archive.cloudera.com/cdh5/parcels/5.16.1/CDH-5.16.1-1.cdh5.16.1.p0.3-el7.parcel.sha1)
  * [manifest.json](http://archive.cloudera.com/cdh5/parcels/5.16.1/manifest.json)

  ```html
  http://archive.cloudera.com/cdh5/parcels/latest/
  ```

#### 基础配置

* 关闭防火墙

* 主机名ip映射

* ssh

* jdk

* 时间同步：master节点作为ntp服务器与外界对时中心同步时间，随后对所有datanode节点提供时间同步服务

  ```shell
  yum install ntp
  # 配置开机启动
  chkconfig ntpd on
  # 检查是否设置成功：其中2-5为on状态就代表成功。
  chkconfig --list ntpd'
  # 在master手动同步一下时间
  ntpdate
  # 设置对时中心
  ntpdate -u 65.55.56.206
  ```

* ntp master服务配置文件

	```shell
	driftfile /var/lib/ntp/drift
	restrict 127.0.0.1
	restrict -6 ::1
	restrict default nomodify notrap 
	server 65.55.56.206 prefer
	includefile /etc/ntp/crypto/pw
	keys /etc/ntp/keys
	```

* 重启master

	```shell
	service ntpd start
	# 用ntpstat命令查看同步状态，出现以下状态代表启动成功
	> ntpstat
	synchronised to NTP server () at stratum 2
	time correct to within 74 ms
	polling server every 128 s
	```

* ntp slave配置文件

	```shell
	driftfile /var/lib/ntp/drift
	restrict 127.0.0.1
	restrict -6 ::1
	restrict default kod nomodify notrap nopeer noquery
	restrict -6 default kod nomodify notrap nopeer noquery
	#这里是主节点的主机名或者ip
	server n1
	includefile /etc/ntp/crypto/pw
	keys /etc/ntp/keys
	```

* 在slave启动ntp服务，手动同步一下时间

	```shell
	service ntpd start
	ntpdate -u n1
	```

	> 一般是本地的ntp服务器还没有正常启动，一般需要等待5-10分钟才可以正常同步

* mysql

  ```mysql
  #hive
  create database hive DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
  #activity monitor
  create database amon DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
  
  #授权root用户在主节点拥有所有数据库的访问权限
  grant all privileges on *.* to 'root'@'n1' identified by 'xxxx' with grant option;
  flush privileges;
  ```

#### 安装Cloudera Manager Server/Agent

* 将cloudera-manager*.tar.gz解压后的cm-5.12.1和cloudera目录放到/opt目录下

* 为Cloudera Manager 5建立数据库，找到mysql-connector-java-5.1.33-bin.jar，放到`/opt/cm-5.12.1/share/cmf/lib/`，在master节点生成数据库

  ```shell
  /opt/cm-5.12.1/share/cmf/schema/scm_prepare_database.sh mysql cm -hlocalhost -uroot -pxxxx --scm-host localhost scm scm scm
  # 参数说明
  /usr/share/cmf/schema/scm_prepare_database.sh [options] <databaseType>  <databaseName>  <databaseUser>  <password>
  --scm-host：安装Cloudera Manager Server的主机名。如果Cloudera Manager Server和数据库安装在同一主机上，请不要使用此选项或-h 选项。
  ```

* 配置Agent，修改`/opt/cm-5.12.1/etc/cloudera-scm-agent/config.ini`中的server_host为主节点的主机名

* 分发cm-5.12.1到slave

* 在所有节点创建cloudera-scm用户组

  ```shell
  useradd --system --home=/opt/cm-5.12.1/run/cloudera-scm-server/ --no-create-home --shell=/bin/false --comment "Cloudera SCM User" cloudera-scm
  ```

* 将CHD5相关的Parcel包放到主节点的`/opt/cloudera/parcel-repo/`目录中（parcel-repo需要手动创建）

  ```shell
  CDH-5.16.1-1.cdh5.16.1.p0.3-el7.parcel
  CDH-5.16.1-1.cdh5.16.1.p0.3-el7.parcel.sha1
  manifest.json
  ```

  > 最后将CDH-5.12.1-1.cdh5.12.1.p0.3-el6.parcel.sha1，重命名为CDH-5.12.1-1.cdh5.12.1.p0.3-el6.parcel.sha，这点必须注意，否则，系统会重新下载CDH-5.12.1-1.cdh5.12.1.p0.3-el6.parcel文件。

* 相关启动脚本

  通过`/opt/cm-5.12.1/etc/init.d/cloudera-scm-server start`启动服务端

  通过`/opt/cm-5.12.1/etc/init.d/cloudera-scm-agent start`启动Agent服务

  > 需要停止服务将以上的start参数改为stop就可以了，重启是restart

#### CDH5的安装配置

* 这时可以通过浏览器访问主节点的7180端口测试一下了（由于CM Server的启动需要花点时间，这里可能要等待一会才能访问），默认的用户名和密码均为admin
* 拷贝mysql驱动到`/opt/cloudera/parcels/CDH-5.12.1-1.cdh5.12.1.p0.3/lib/hive/lib/`下

#### 集群测试

```shell
sudo -u hdfs hadoop jar /opt/cloudera/parcels/CDH/lib/hadoop-mapreduce/hadoop-mapreduce-examples.jar pi 10 100
```

* 安装参考`https://blog.csdn.net/gtsina/article/details/78048925`

* HA参考`https://blog.csdn.net/post_yuan/article/details/78626340`、`https://blog.csdn.net/freedomboy319/article/details/46357495`


### 伪分布

* hadoop-env.sh，修改JAVA_HOME

  ```shell
  export JAVA_HOME=/opt/module/jdk1.8.0_144
  ```

* core-site.xml，配置NameNode地址，配置hadoop临时目录

  ```xml
  <!-- 指定HDFS中NameNode的地址 -->
  <property>
  <name>fs.defaultFS</name>
      <value>hdfs://hadoop101:9000</value>
  </property>
  
  <!-- 指定Hadoop运行时产生文件的存储目录 -->
  <property>
  	<name>hadoop.tmp.dir</name>
  	<value>/opt/module/hadoop-2.7.2/data/tmp</value>
  </property>
  ```

* hdfs-site.xml，配置伪分布环境下的副本数

  ```xml
  <!-- 指定HDFS副本的数量 -->
  <property>
  	<name>dfs.replication</name>
  	<value>1</value>
  </property>
  ```

* 格式化namenode，第一次启动hdfs前需要

  ```shell
   bin/hdfs namenode -format
  ```

* 启动，分别启动NameNode、DataNode

  ```shell
  sbin/hadoop-daemon.sh start namenode
  sbin/hadoop-daemon.sh start datanode
  ```

* yarn-evn.sh，修改JAVA_HOME

  ```shell
  export JAVA_HOME=/opt/module/jdk1.8.0_144
  ```

* yarn-site.xml，配置Reducer从Map端的获取方式，皮遏制RM地址

  ```xml
  <!-- Reducer获取数据的方式 -->
  <property>
      <name>yarn.nodemanager.aux-services</name>
      <value>mapreduce_shuffle</value>
  </property>
  
  <!-- 指定YARN的ResourceManager的地址 -->
  <property>
      <name>yarn.resourcemanager.hostname</name>
      <value>hadoop101</value>
  </property>
  ```

* yarn-site.xml，配置日志聚集，需要重启NodeManager、ResourceManager、HistoryManager节点

  ```xml
  <!-- 日志聚集功能使能 -->
  <property>
  <name>yarn.log-aggregation-enable</name>
  <value>true</value>
  </property>
  
  <!-- 日志保留时间设置7天 -->
  <property>
  <name>yarn.log-aggregation.retain-seconds</name>
  <value>604800</value>
  </property>
  ```

* mapred-env.sh，配置JAVA_HOME

  ```shell
  export JAVA_HOME=/opt/module/jdk1.8.0_144
  ```

* mapred-site.xml，将mapred-site.xml.template重命名

  ```xml
  <!-- 指定MR运行在YARN上 -->
  <property>
      <name>mapreduce.framework.name</name>
      <value>yarn</value>
  </property>
  ```

* mapred-site.xml，配置历史服务器

  ```xml
  <!-- 历史服务器端地址 -->
  <property>
  <name>mapreduce.jobhistory.address</name>
  <value>hadoop101:10020</value>
  </property>
  
  <!-- 历史服务器web端地址 -->
  <property>
      <name>mapreduce.jobhistory.webapp.address</name>
      <value>hadoop101:19888</value>
  </property>
  ```

* 启动，必须先启动NameNode、DateNode

  ```shell
  sbin/yarn-daemon.sh start resourcemanager
  sbin/yarn-daemon.sh start nodemanager
  sbin/mr-jobhistory-daemon.sh start historyserver
  ```

### 分布式

#### 节点规划

|      | node1             | node2                      | node3                       |
| ---- | ----------------- | -------------------------- | --------------------------- |
| HDFS | NameNode/DataNode | SecondaryNameNode/DataNode | DataNode                    |
| YARN | NodeManager       | NodeManager                | ResourceManager/NodeManager |

#### 核心配置

* hadoop-env.sh，配置java环境变量 

  ```xml
  export JAVA_HOME=/usr/local/jdk
  ```

* core-site.xml，配置NameNode节点地址，hadoop运行时目录位置

  ```xml
  <!-- 指定HDFS中NameNode的地址 -->
  <property>
  		<name>fs.defaultFS</name>
        <value>hdfs://node1:9000</value>
  </property>
  
  <!-- 指定Hadoop运行时产生文件的存储目录 -->
  <property>
  		<name>hadoop.tmp.dir</name>
  		<value>/data/hadoop/tmp</value>
  </property>
  ```

* 配置slaves，用于集群启动

  ```shell
  node1
  node2
  node3
  ```

#### HDFS配置

* hdfs-site.xml，配置副本数（默认是3），配置SecondaryNameNode节点地址

  ```xml
  <property>
  		<name>dfs.replication</name>
  		<value>3</value>
  </property>
  
  <!-- 指定Hadoop辅助名称节点主机配置 -->
  <property>
        <name>dfs.namenode.secondary.http-address</name>
        <value>node2:50090</value>
  </property>
  ```

#### YARN配置

* yarn-env.sh，配置java环境变量

  ```xml
  export JAVA_HOME=/opt/module/jdk1.8.0_144
  ```

* yarn-site.xml，配置reduce阶段获取数据方式，配置ResourceManager节点地址

  ```xml
  <!-- Reducer获取数据的方式 -->
  <property>
  		<name>yarn.nodemanager.aux-services</name>
  		<value>mapreduce_shuffle</value>
  </property>
  
  <!-- 指定YARN的ResourceManager的地址 -->
  <property>
  		<name>yarn.resourcemanager.hostname</name>
  		<value>node3</value>
  </property>
  ```

#### MapReduce配置

* mapred-env.sh，配置java环境变量

  ```xml
  export JAVA_HOME=/opt/module/jdk1.8.0_144
  ```

* mapred-site.xml，配置MR任务提交到yarn运行

  ```xml
  <!-- 指定MR运行在Yarn上 -->
  <property>
      <name>mapreduce.framework.name</name>
      <value>yarn</value>
  </property>
  ```

#### 格式化集群

```shell
bin/hdfs namenode -format
```

####  启动集群

* 启动集群，必须配置slaves节点

  ```shell
  # 启动hdfs，必须在NameNode节点
  start-dfs.sh
  # 启动yarn，必须在ResourceManager节点
  start-yarn.sh
  ```

* 查看集群启动

  ```http
  http://node1:50070 //hdfs
  http://node2:50090 //SecondaryNameNode
  http://node3:8088 //yarn
  http://192.168.56.14:19888/jobhistory //HistoryManager
  ```


### 分布式HA

#### 节点规划

| node1       | node2       | node3       | node4              | node5              | node6       |
| ----------- | ----------- | ----------- | ------------------ | ------------------ | ----------- |
| Zookeeper   | Zookeeper   | Zookeeper   |                    |                    |             |
|             |             |             | NameNode(A)        | NameNode(S)        |             |
|             |             |             | JournalNode        | JournalNode        |             |
|             |             |             | ResourceManager(A) | ResourceManager(S) |             |
| DataNode    | DataNode    | DataNode    |                    |                    | DataNode    |
| NodeManager | NodeManager | NodeManager |                    |                    | NodeManager |
|             |             |             | HistoryManager     |                    |             |

#### mapred-site.xml

```xml
<property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
</property>
<!-- 历史服务器端地址 -->
<property>
    <name>mapreduce.jobhistory.address</name>
    <value>node4:10020</value>
</property>
<!-- 历史服务器web端地址 -->
<property>
    <name>mapreduce.jobhistory.webapp.address</name>
    <value>node4:19888</value>
</property>
```

#### core-site.xml

```xml
<configuration>
    <!-- 把两个NameNode）的地址组装成一个集群mycluster -->
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://mycluster</value>
    </property>

    <!-- 指定hadoop运行时产生文件的存储目录 -->
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/data/hadoopha/tmp</value>
    </property>
    
    <!-- 配置zk地址 -->
    <property>
        <name>ha.zookeeper.quorum</name>
        <value>node1:2181,node2:2181,node3:2181</value>
    </property>
</configuration>
```

#### hdfs-site.xml

```xml
<configuration>
    <!-- 文件副本数 -->
    <property>
        <name>dfs.replication</name>
        <value>3</value>
    </property>
    <!-- 完全分布式集群名称 -->
    <property>
        <name>dfs.nameservices</name>
        <value>mycluster</value>
    </property>

    <!-- 集群中NameNode节点都有哪些 -->
    <property>
        <name>dfs.ha.namenodes.mycluster</name>
        <value>nn1,nn2</value>
    </property>

    <!-- nn1的RPC通信地址，官网docs使用的端口时8020 -->
    <property>
        <name>dfs.namenode.rpc-address.mycluster.nn1</name>
        <value>node4:9000</value>
    </property>

    <!-- nn2的RPC通信地址 -->
    <property>
        <name>dfs.namenode.rpc-address.mycluster.nn2</name>
        <value>node5:9000</value>
    </property>

    <!-- nn1的http通信地址 -->
    <property>
        <name>dfs.namenode.http-address.mycluster.nn1</name>
        <value>node4:50070</value>
    </property>

    <!-- nn2的http通信地址 -->
    <property>
        <name>dfs.namenode.http-address.mycluster.nn2</name>
        <value>node5:50070</value>
    </property>

    <!-- 指定NameNode元数据在JournalNode上的存放位置 -->
    <property>
        <name>dfs.namenode.shared.edits.dir</name>
        <value>qjournal://node4:8485;node5:8485/mycluster</value>
    </property>

    <!-- 配置隔离机制，即同一时刻只能有一台服务器对外响应 -->
    <property>
        <name>dfs.ha.fencing.methods</name>
        <value>sshfence</value>
    </property>
    <!-- 连接超时时间 -->
    <property>
        <name>dfs.ha.fencing.ssh.connect-timeout</name>
        <value>30000</value>
    </property>

    <!-- 使用隔离机制时需要ssh无秘钥登录-->
    <property>
        <name>dfs.ha.fencing.ssh.private-key-files</name>
        <value>/home/root/.ssh/id_rsa</value>
    </property>

    <!-- 声明journalnode服务器存储目录-->
    <property>
        <name>dfs.journalnode.edits.dir</name>
        <value>/data/hadoopha/jn</value>
    </property>

    <!-- 关闭权限检查，生产环境建议开启，针对不同用户做权限检查-->
    <property>
        <name>dfs.permissions.enable</name>
        <value>false</value>
    </property>

    <!-- 访问代理类：client，mycluster，active配置失败自动切换实现方式-->
    <property>
        <name>dfs.client.failover.proxy.provider.mycluster</name>
        <value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
    </property>
    <!-- 开启HA自动故障转移 -->
    <property>
        <name>dfs.ha.automatic-failover.enabled</name>
        <value>true</value>
    </property>
</configuration>
```

#### yarn-site.xml

```xml
<configuration>
    <!-- 日志聚集功能使能 -->
    <property>
        <name>yarn.log-aggregation-enable</name>
        <value>true</value>
    </property>

    <!-- 日志保留时间设置7天 -->
    <property>
        <name>yarn.log-aggregation.retain-seconds</name>
        <value>604800</value>
    </property>

    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>

    <!--启用resourcemanager ha-->
    <property>
        <name>yarn.resourcemanager.ha.enabled</name>
        <value>true</value>
    </property>

    <!--声明两台resourcemanager的地址-->
    <property>
        <name>yarn.resourcemanager.cluster-id</name>
        <value>cluster-yarn</value>
    </property>
	
    <!-- 指定ResourceManager名称 -->
    <property>
        <name>yarn.resourcemanager.ha.rm-ids</name>
        <value>rm1,rm2</value>
    </property>
	
    <!-- 指定rm1主机 -->
    <property>
        <name>yarn.resourcemanager.hostname.rm1</name>
        <value>node4</value>
    </property>

    <!-- 指定rm2主机 -->
    <property>
        <name>yarn.resourcemanager.hostname.rm2</name>
        <value>node5</value>
    </property>

    <!--指定zookeeper集群的地址--> 
    <property>
        <name>yarn.resourcemanager.zk-address</name>
        <value>node1:2181,node2:2181,node3:2181</value>
    </property>

    <!--启用自动恢复--> 
    <property>
        <name>yarn.resourcemanager.recovery.enabled</name>
        <value>true</value>
    </property>

    <!--指定resourcemanager的状态信息存储在zookeeper集群--> 
    <property>
        <name>yarn.resourcemanager.store.class</name>     <value>org.apache.hadoop.yarn.server.resourcemanager.recovery.ZKRMStateStore</value>
    </property>
</configuration>
```

#### slaves

```xml
<!-- 指定DataNode节点主机，只用于启动集群 -->
node1
node2
node3
```

#### 启动集群

* 在各个节点启动JournalNode

  ```shell
  sbin/hadoop-daemon.sh start journalnode
  ```

* 在`nn1`上格式化hdfs

  ```shell
  bin/hdfs namenode -format
  ```

* 在`nn1`上启动NameNode

  ```shell
  sbin/hadoop-daemon.sh start namenode
  ```

* 在`nn2`同步`nn1`的 数据

  ```shell
    bin/hdfs namenode -bootstrapStandby
  ```

* 在`nn2`上启动NameNode

  ```shell
  sbin/hadoop-daemon.sh start namenode
  ```

* 启动zk，初始化HA在zk中的状态

  ```shell
  bin/zkServer.sh start
  # 在NameNode节点
  bin/hdfs zkfc -formatZK
  ```

* 启动hdfs

  ```shell
  sbin/start-dfs.sh
  ```

* 在各个NameNode节点启动DFSZK Failover Controller 

  ```shell
  sbin/hadoop-daemin.sh start zkfc
  ```

* 在node4上启动yarn

  ```shell
  sbin/start-yarn.sh
  ```

* 在node5上单独启动ResourceManager

  ```shell
  sbin/yarn-daemon.sh start resourcemanager
  ```

* 查看yarn状态

  ```shell
  bin/yarn rmadmin -getServiceState rm1
  ```

* 启动历史服务器

  ```shell
  sbin/mr-jobhistory-daemon.sh start/stop historyserver
  ```

#### 问题

* standby无法切换到active状态，查看日志没有fuser命令，安装yum install psmisc

  ```shell
  2018-11-25 20:39:42,427 WARN org.apache.hadoop.ha.SshFenceByTcpPort: PATH=$PATH:/sbin:/usr/sbin fuser -v -k -n tcp 9000 via ssh: bash: fuser: 未找到命令
  ```


## HDFS操作

## MapReduce

### WC编写

#### pom

```xml
<dependencies>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-core</artifactId>
        <version>2.8.2</version>
    </dependency>
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-common</artifactId>
        <version>2.7.7</version>
    </dependency>
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-client</artifactId>
        <version>2.7.7</version>
    </dependency>
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-hdfs</artifactId>
        <version>2.7.7</version>
    </dependency>
</dependencies>
```

#### 日志文件

```properties
log4j.rootLogger=INFO, stdout
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%d %p [%c] - %m%n
log4j.appender.logfile=org.apache.log4j.FileAppender
log4j.appender.logfile.File=target/spring.log
log4j.appender.logfile.layout=org.apache.log4j.PatternLayout
log4j.appender.logfile.layout.ConversionPattern=%d %p [%c] - %m%n
```

#### 运行

* 本地运行需要配置HADOOP_HOME

* 提交到集群运行需要打包

  ```xml
  <build>
      <plugins>
          <plugin>
              <artifactId>maven-compiler-plugin</artifactId>
              <version>2.3.2</version>
              <configuration>
                  <source>1.8</source>
                  <target>1.8</target>
              </configuration>
          </plugin>
          <plugin>
              <artifactId>maven-assembly-plugin </artifactId>
              <configuration>
                  <descriptorRefs>
                      <descriptorRef>jar-with-dependencies</descriptorRef>
                  </descriptorRefs>
                  <archive>
                      <manifest>
                          <mainClass>com.foo.WordcountDriver</mainClass>
                      </manifest>
                  </archive>
              </configuration>
              <executions>
                  <execution>
                      <id>make-assembly</id>
                      <phase>package</phase>
                      <goals>
                          <goal>single</goal>
                      </goals>
                  </execution>
              </executions>
          </plugin>
      </plugins>
  </build>
  ```

#### windows下运行





### FileInputFormat实现类 

#### TextInputFormat

* FileInputFormat默认实现类
* 按行读取数据
* key是LongWritable类型，表示该行起始位置在整个文件中的偏移量
* value是当前行内容，不包含任何换行符、回车符

#### KeyValueTextInputFormat

* 每一行为一条记录
* key是被分割符分割的第一个数据，index[0]
* value是被分割符分割的其他数据，index[1-->]

#### NLineInputFormat

* map处理的InputSplit不在按照block划分，分片数按照指定的行数划分
* key是LongWritable类型，表示该行起始位置在整个文件中的偏移量
* value是当前行内容，不包含任何换行符、回车符

### CombineTextInputFormat切片机制 

* 计算方式

  ```java
  虚拟存储 4M
  	if(f>4M,f<4M*2)
  		4M/2
  	10M=4+3+3
  ```

* 程序设置

  ```java
  // 如果不设置InputFormat，它默认用的是TextInputFormat.class
  job.setInputFormatClass(CombineTextInputFormat.class);
  //虚拟存储切片最大值设置4m
  CombineTextInputFormat.setMaxInputSplitSize(job, 4194304);
  ```





Maptask

​	环形缓冲区-快排（分区）

​	写磁盘-归并

Reducetask

​	从map获取数据

​	内存（分区）

​	磁盘-归并（Grouping）

​	reduce