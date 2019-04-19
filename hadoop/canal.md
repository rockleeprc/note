## 单实例

### MySQL准备

* 配置开启binlog

```shell
[mysqld]
log-bin=mysql-bin #添加这一行就ok
binlog-format=ROW #选择row模式
server_id=1 #配置mysql replaction需要定义，不能和canal的slaveId重复
```

* canal授权

  ```sql
  CREATE USER canal IDENTIFIED BY 'canal';  
  GRANT SELECT, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'canal'@'%';
  -- GRANT ALL PRIVILEGES ON *.* TO 'canal'@'%' ;
  FLUSH PRIVILEGES;
  ```

### Canal配置

* `$canal_home/conf/example/instance.properties`

  ```properties
  # 数据库实例地址
  canal.instance.master.address=
  # 数据库用户名
  canal.instance.dbUsername=
  # 用户和密码
  canal.instance.dbPassword=
  # 连接字符集编码
  canal.instance.connectionCharset = UTF-8
  # 默认连接数据库
  canal.instance.defaultDatabaseName = 
  ```

### 启动

```shell
$canal_home/bin/startup.sh
```

### 日志

* `$canal_home/logs/canal`canal自己日志
* `$canal_home/logs/example` canna配置实例日志

## Canal Kafka

### 配置

* `$canal_home/conf/example/instance.properties` 数据配置信息见`单实例`

  ```properties
  canal.mq.topic=example
  canal.mq.partition=0
  ```

* `$canal_home/conf/canal.properties`

  ```properties
  canal.serverMode = kafka
  ```

###  Canal HA

### 配置

* `$canal_home/conf/canal.properties`

  ```properties
  canal.zkServers = node1:2181,node2:2181,node3:2181
  canal.instance.global.spring.xml = classpath:spring/default-instance.xml
  ```

* `$canal_home/conf/example/instance.properties`，两台机器上的instance目录的名字需要保证完全一致

  ```properties
  # 另外一台机器改成1235，保证slaveId不重复即可
  canal.instance.mysql.slaveId = 1234 
  ```

### 启动

* 在两个节点启动canal

* zk中查看节点状态

  ```shell
  [zk: localhost:2181(CONNECTED) 49] get /otter/canal/destinations/example/running
  {"active":true,"address":"192.168.13.213:11111","cid":1}
  ```

  