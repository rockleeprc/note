

集群规划
node1
    DataNode
    NodeManager
    NameNode
    JobHistory
node2
    DataNode
    NodeManager
    ResourceManager
node3
    DataNode
    NodeManager
    SecondaryNameNode

core-site.xml
```xml
<configuration>
    <!-- 指定 NameNode 的地址 --> 
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://node1:8020</value> 
    </property>
    <!-- 指定 hadoop 数据的存储目录 -->
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/usr/local/hadoop/data</value>
    </property>
    <!-- 配置 HDFS 网页登录使用的静态用户为 root --> 
    <property>
        <name>hadoop.http.staticuser.user</name>
        <value>root</value> 
    </property>
</configuration>
```

hdfs-site.xml
```xml
<configuration>
    <!-- nn web端访问地址--> 
    <property>
        <name>dfs.namenode.http-address</name>
        <value>node1:9870</value> 
    </property>
    <!-- 2nn web 端访问地址--> 
    <property>
        <name>dfs.namenode.secondary.http-address</name>
        <value>node3:9868</value> 
    </property>
    <!-- 设置hdfs文件副本数-->
    <property>
        <name>dfs.replication</name>
        <value>2</value>
    </property>
</configuration>
```

yarn-site.xml
```xml
<configuration>
    <!-- 指定 MR 走 shuffle --> 
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value> 
    </property>
    <!-- 指定 ResourceManager 的地址--> 
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>node2</value> 
    </property> 
    <!-- 开启日志聚集功能 --> 
    <property>
        <name>yarn.log-aggregation-enable</name>
        <value>true</value>
    </property>
    <!-- 设置日志聚集服务器地址 --> 
    <property>
        <name>yarn.log.server.url</name>
        <value>http://node1:19888/jobhistory/logs</value>
    </property>
    <!-- 设置日志保留时间为 7 天 --> 
    <property>
        <name>yarn.log-aggregation.retain-seconds</name>
        <value>604800</value>
    </property>
    <!-- 环境变量的继承 -->
    <property> 
        <name>yarn.nodemanager.env-whitelist</name>
        <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
   </property>
</configuration>
```

mapred-site.xml
```xml
<configuration>
    <!-- 指定 MapReduce 程序运行在 Yarn 上 --> 
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
    <!-- 历史服务器端地址 --> 
    <property>
        <name>mapreduce.jobhistory.address</name>
        <value>node1:10020</value> 
    </property>
    <!-- 历史服务器 web 端地址 --> 
    <property>
        <name>mapreduce.jobhistory.webapp.address</name>
        <value>node1:19888</value> 
    </property>
</configuration>
```

HADOOP_HOME/etc/hadoop/workers
node1
node2
node3
 
 集群启动
    如果集群是第一次启动，需要在 node1 节点格式化 NameNode(注意:格式化 NameNode，会产生新的集群 id，导致 NameNode 和 DataNode 的集群 id 不一致，集群找 不到已往数据。如果集群在运行过程中报错，需要重新格式化 NameNode 的话，一定要先停 止 namenode 和 datanode 进程，并且要删除所有机器的 data 和 logs 目录，然后再进行格式化。)

无法识别JAVA_HOME，在hadoop-env.sh配置JAVA_HOME

root用户无法启动集群，修改start-dfs.sh，stop-dfs.sh
```shell
HDFS_DATANODE_USER=root
HDFS_DATANODE_SECURE_USER=hdfs
HDFS_NAMENODE_USER=root
HDFS_SECONDARYNAMENODE_USER=root
```
start-yarn.sh，stop-yarn.sh
```shell
YARN_RESOURCEMANAGER_USER=root
HDFS_DATANODE_SECURE_USER=yarn
YARN_NODEMANAGER_USER=root
```
```shell
#!/bin/bash
if [ $# -lt 1 ]
then
   echo "No Args Input..."
exit ; 
fi
case $1 in "start")
echo " =================== 启动 hadoop 集群 ==================="
echo " --------------- 启动 hdfs ---------------"
ssh node1 "/usr/local/hadoop/sbin/start-dfs.sh" 
echo " --------------- 启动 yarn ---------------"
ssh node2 "/usr/local/hadoop/sbin/start-yarn.sh"
echo " --------------- 启动 historyserver ---------------"
ssh node1 "/usr/local/hadoop/bin/mapred --daemon start historyserver"
;;
*)
   echo "Input Args Error..."
;;
esac
```

集群启动
```shell
# node1格式化namenode
$ bin/hdfs namenode -format
# node1 集群启动hdfs
$ sbin/start-dfs.sh/stop-dfs.sh
# node2 集群启动yarn
$ sbin/start-yarn.sh/stop-yarn.sh
# node1 启动jobhistory
$ bin/mapred --daemon start historyserver
```
单独启动
```shell
$ hdfs --daemon start/stop namenode/datanode/secondarynamenode
$ yarn --daemon start/stop resourcemanager/nodemanager
```

Web 端查看 HDFS 的 NameNode
    http://node1:9870
Web 端查看 YARN 的 ResourceManager
    http://node2:8088
web 历史服务器地址
    http://node1:19888/jobhistory



hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.4.jar wordcount input wcoutput
 

## hdfs 写入流程

## hdfs 读取流程

## 