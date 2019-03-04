## 安装

### 集群

* 配置文件目录$spark_home/conf 

* slaves中加入savle主机名

  ```shell
  node1
  node2
  node3
  node4
  ```

* spark-env.sh，配置master主机名、端口、历史服务器启动参数

  ```shell
  SPARK_MASTER_HOST=node1
  SPARK_MASTER_PORT=7077
  export JAVA_HOME=/usr/local/jdk
  export SCALA_HOME=/usr/local/scala
  export HADOOP_HOME=/usr/local/hadoop
  export SPARK_HISTORY_OPTS="-Dspark.history.ui.port=4000 -Dspark.history.retainedApplications=3 -Dspark.history.fs.logDirectory=hdfs://node1:9000/directory"
  ```

* spark-default.conf，配置历史服务器

  ```shell
  spark.eventLog.enabled  true
  spark.eventLog.dir       hdfs://node1:9000/directory
  spark.eventLog.compress true
  ```

* 将配置分发到其它节点

* 启动spark $spark_home/sbin/start-all.sh

* 启动spark历史服务器 $spark_home/sbin/start-history-server.sh

* 权限问题在hdfs-site.xml中配置关闭权限

  ```shell
  <property>
      <name>dfs.permissions</name>
      <value>false</value>
  </property>  
  ```

* 提交任务测试集群

  ```spark
  $spark_home/bin/spark-submit \
  --class org.apache.spark.examples.SparkPi \
  --master spark://master01:7077 \
  --executor-memory 1G \
  --total-executor-cores 2 \
  /$spark_home/examples/jars/spark-examples_2.11-2.1.1.jar \
  100
  
  ```

  