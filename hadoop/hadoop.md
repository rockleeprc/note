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
192.168.56.11 node1
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

| node1       | node2       | node3       | node4              | node5              | node6 |
| ----------- | ----------- | ----------- | ------------------ | ------------------ | ----- |
| Zookeeper   | Zookeeper   | Zookeeper   |                    |                    |       |
|             |             |             | NameNode(A)        | NameNode(S)        |       |
|             |             |             | JournalNode        | JournalNode        |       |
|             |             |             | ResourceManager(A) | ResourceManager(S) |       |
| DataNode    | DataNode    | DataNode    |                    |                    |       |
| NodeManager | NodeManager | NodeManager |                    |                    |       |
|             |             |             | HistoryManager     |                    |       |

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

    <!-- nn1的RPC通信地址 -->
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

    <!-- 关闭权限检查-->
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

    <property>
        <name>yarn.resourcemanager.ha.rm-ids</name>
        <value>rm1,rm2</value>
    </property>

    <property>
        <name>yarn.resourcemanager.hostname.rm1</name>
        <value>node2</value>
    </property>

    <property>
        <name>yarn.resourcemanager.hostname.rm2</name>
        <value>node3</value>
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

*  在`nn2`同步`nn1`的 数据

  ```shell
  bin/hdfs namenode -bootstrapStandby
  ```

* 在`nn2`上启动NameNode

  ```shell
  sbin/hadoop-daemon.sh start namenode
  ```

---

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

* 在node2上启动yarn

  ```shell
  sbin/start-yarn.sh
  ```

* 在node3上单独启动ResourceManager

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

```shell
2018-11-25 20:39:42,427 WARN org.apache.hadoop.ha.SshFenceByTcpPort: PATH=$PATH:/sbin:/usr/sbin fuser -v -k -n tcp 9000 via ssh: bash: fuser: 未找到命令
```

* 安装yum install psmisc



## HDFS操作

## MR

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
                          <mainClass>com.atguigu.mr.WordcountDriver</mainClass>
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

```java
// 如果不设置InputFormat，它默认用的是TextInputFormat.class
job.setInputFormatClass(CombineTextInputFormat.class);

//虚拟存储切片最大值设置4m
CombineTextInputFormat.setMaxInputSplitSize(job, 4194304);
```



