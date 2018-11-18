## Linux准备

### 安装JDK

* 略

### 创建hadoop用户/租/分配root权限

* 添加hadoop组

  ```shell
  groupadd group
  ```

* 添加hadoop用户

  ```shell
  groupadd mysql
  ```

* 将hadoop用户添加到hadoop组、

  ```shell
  useradd mysql -g mysql
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

  禁止firewallk开机启动

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
192.168.0.211 hadoop001
192.168.0.212 hadoop002
192.168.0.213 hadoop003
# 将修改hosts拷贝到其它节点
scp  –r /etc/hosts  hadoop@192.168.0.212：/etc/
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

* 自动

  ```shell
  zkServer.sh status
  ZooKeeper JMX enabled by default
  Using config: /usr/local/zookeeper/bin/../conf/zoo.cfg
  Mode: leader
  ```

  



## Hadoop安装

* ### 环境规划

  - /opt/modul，存放所有解压后jar
  - /opt/software，存放下载*.jar
  - 所有安装都使用hadoop用户

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

* core-site.xml

  ```xml
  <!-- 指定HDFS中NameNode的地址 -->
  <property>
  		<name>fs.defaultFS</name>
        <value>hdfs://hadoop102:9000</value>
  </property>
  
  <!-- 指定Hadoop运行时产生文件的存储目录 -->
  <property>
  		<name>hadoop.tmp.dir</name>
  		<value>/opt/module/hadoop-2.7.2/data/tmp</value>
  </property>
  ```

#### HDFS配置

* hadoop-env.sh 

  ```xml
  export JAVA_HOME=/opt/module/jdk1.8.0_144
  ```

* hdfs-site.xml 

  ```xml
  <property>
  		<name>dfs.replication</name>
  		<value>3</value>
  </property>
  
  <!-- 指定Hadoop辅助名称节点主机配置 -->
  <property>
        <name>dfs.namenode.secondary.http-address</name>
        <value>hadoop104:50090</value>
  </property>
  ```

#### YARN配置

* yarn-env.sh

  ```xml
  export JAVA_HOME=/opt/module/jdk1.8.0_144
  ```

* yarn-site.xml

  ```xml
  <!-- Reducer获取数据的方式 -->
  <property>
  		<name>yarn.nodemanager.aux-services</name>
  		<value>mapreduce_shuffle</value>
  </property>
  
  <!-- 指定YARN的ResourceManager的地址 -->
  <property>
  		<name>yarn.resourcemanager.hostname</name>
  		<value>hadoop103</value>
  </property>
  
  ```

#### MapReduce配置

* mapred-env.sh 

  ```xml
  export JAVA_HOME=/opt/module/jdk1.8.0_144
  ```

* mapred-site.xml

  ```xml
  <!-- 指定MR运行在Yarn上 -->
  <property>
  		<name>mapreduce.framework.name</name>
  		<value>yarn</value>
  </property>
  
  ```

####  启动集群

* 配置slaves，用于集群启动

  ```shell
  node1
  node2
  node3
  ```

* 启动集群

  ```shell
  # 启动hdfs，必须在NameNode节点
  start-dfs.sh
  # 启动yarn，不惜在ResourceManager节点
  start-yarn.sh
  ```

* 查看集群启动

  ```http
  http://node1:500770 //hdfs
  http://node2:50090/status.html //SecondaryNameNode
  ```

  

### 分布式HA

## HDFS操作



