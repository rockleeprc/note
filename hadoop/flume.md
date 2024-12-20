## 安装

* 修改`flume_home/conf/flume-env.sh.template`文件名，以及文件中的`JAVA_HOME`路径

## Flume内部原理

### put事物流程

1. doPut：将数据写入临时缓冲区putList
2. doCommit：检查Channel内存队列是否足够合并
3. doRollback：Channel内存队列空间不足，回滚数据到putList中

### take事物流程

1. doTake：从Channel中将数据取到临时缓冲区takeList中
2. doCommit：如果数据全部发送Sink成功，清除takeList中的数据
3. doRollback：数据发送过程中出现异常，将takeList中的数据退回给Channel

## 案例

> * flume采集数据到hdfs，要先保证flume有hdfs相关的jar包
>   1. commons-configuration-1.6.jar
>   2. hadoop-auth-2.7.2.jar
>   3. hadoop-common-2.7.2.jar
>   4. hadoop-hdfs-2.7.2.jar
>   5. commons-io-2.4.jar
>   6. htrace-core-3.1.0-incubating.jar
> * 拷贝jar包$sqoop_home/lib下
> * 启动flume方式
>   1. $ bin/flume-ng agent --conf conf/ --name a2 --conf-file job/flume-file-hdfs.conf
>   2. agent：flume的一个组件，必须使用agent
>   3. --conf conf/：flume的配置文件
>   4. --name a2：自定义agent名称
>   5. --conf-file job/flume-file-hdfs.conf：自定义配置文件

### 实时读取本地文件到hdfs

```shell
# 定义agent、sources、sinks、channels的名称
a2.sources = r2
a2.sinks = k2
a2.channels = c2

# sources配置
# exec为可执行命令
a2.sources.r2.type = exec
# 具体的命令
a2.sources.r2.command = tail -F /opt/module/hive/logs/hive.log
# 执行shell脚本的绝对路径
a2.sources.r2.shell = /bin/bash -c

# 配置sinks
a2.sinks.k2.type = hdfs
# hdfs路径，目录自动生成
a2.sinks.k2.hdfs.path = hdfs://hadoop102:9000/flume/%Y%m%d/%H
# 上传文件的前缀
a2.sinks.k2.hdfs.filePrefix = logs-
# 是否按照时间滚动文件夹
a2.sinks.k2.hdfs.round = true
# 多少时间单位创建一个新的文件夹
a2.sinks.k2.hdfs.roundValue = 1
# 重新定义时间单位
a2.sinks.k2.hdfs.roundUnit = hour
# 是否使用本地时间戳
a2.sinks.k2.hdfs.useLocalTimeStamp = true
# 积攒多少个Event才flush到HDFS一次
a2.sinks.k2.hdfs.batchSize = 1000
# 设置文件类型，可支持压缩
a2.sinks.k2.hdfs.fileType = DataStream
# 多久生成一个新的文件，时间单位秒
a2.sinks.k2.hdfs.rollInterval = 600
# 设置每个文件的滚动大小，128M
a2.sinks.k2.hdfs.rollSize = 134217700
# 文件的滚动与Event数量无关，数量设置为0
a2.sinks.k2.hdfs.rollCount = 0
# 最小冗余数，副本数
a2.sinks.k2.hdfs.minBlockReplicas = 1

# 配置channels
# 使用内存
a2.channels.c2.type = memory
# 容量为1000个Event
a2.channels.c2.capacity = 1000
# 100条Event后提交事务
a2.channels.c2.transactionCapacity = 100

# 绑定sources、channels、sinks
a2.sources.r2.channels = c2
a2.sinks.k2.channel = c2
```

### 实时读取目录到hdfs

```shell
# 定义agent、sources、sinks、channels的名称
a3.sources = r3
a3.sinks = k3
a3.channels = c3

# 配置sources
# sources类型为spooldir，默认每500毫秒扫描一次目录
a3.sources.r3.type = spooldir
# 监控的目录
a3.sources.r3.spoolDir = /opt/module/flume/upload
# 文件上传完的后缀
a3.sources.r3.fileSuffix = .COMPLETED
# 是否有文件头，拦截器使用文件头过滤数据
a3.sources.r3.fileHeader = true
# 忽略所有以.tmp结尾的文件，不上传
a3.sources.r3.ignorePattern = ([^ ]*\.tmp)

# 配置sinks
a3.sinks.k3.type = hdfs
a3.sinks.k3.hdfs.path = hdfs://hadoop102:9000/flume/upload/%Y%m%d/%H
# 上传文件的前缀
a3.sinks.k3.hdfs.filePrefix = upload-
# 是否按照时间滚动文件夹
a3.sinks.k3.hdfs.round = true
# 多少时间单位创建一个新的文件夹
a3.sinks.k3.hdfs.roundValue = 1
# 重新定义时间单位
a3.sinks.k3.hdfs.roundUnit = hour
# 是否使用本地时间戳
a3.sinks.k3.hdfs.useLocalTimeStamp = true
# 积攒多少个Event才flush到HDFS一次
a3.sinks.k3.hdfs.batchSize = 100
# 设置文件类型，可支持压缩
a3.sinks.k3.hdfs.fileType = DataStream
# 多久生成一个新的文件
a3.sinks.k3.hdfs.rollInterval = 600
# 设置每个文件的滚动大小大概是128M
a3.sinks.k3.hdfs.rollSize = 134217700
# 文件的滚动与Event数量无关
a3.sinks.k3.hdfs.rollCount = 0
# 最小冗余数
a3.sinks.k3.hdfs.minBlockReplicas = 1

# 配置channels
a3.channels.c3.type = memory
a3.channels.c3.capacity = 1000
a3.channels.c3.transactionCapacity = 100

# 绑定sources、channels、sinks
a3.sources.r3.channels = c3
a3.sinks.k3.channel = c3
```

### 单sources多channels/sinks

```shell
# 定义sources、channels、sinks
a1.sources = r1
a1.sinks = k1 k2
a1.channels = c1 c2
# 将数据流复制给所有channel
a1.sources.r1.selector.type = replicating

# Describe/configure the source
a1.sources.r1.type = exec
a1.sources.r1.command = tail -F /opt/module/hive/logs/hive.log
a1.sources.r1.shell = /bin/bash -c

# sinks类型是用avro，对应另一个agent sources
a1.sinks.k1.type = avro
# 可以配置ip或主机名
a1.sinks.k1.hostname = hadoop102 
a1.sinks.k1.port = 4141

a1.sinks.k2.type = avro
a1.sinks.k2.hostname = hadoop102
a1.sinks.k2.port = 4142

# Describe the channel
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

a1.channels.c2.type = memory
a1.channels.c2.capacity = 1000
a1.channels.c2.transactionCapacity = 100

# 一个sources绑定两个channels，两个channels分别绑定两个sinks
# 两个sinks对应不同agent sources
a1.sources.r1.channels = c1 c2
a1.sinks.k1.channel = c1
a1.sinks.k2.channel = c2
```



```shell
# Name the components on this agent
a2.sources = r1
a2.sinks = k1
a2.channels = c1

# sources类型avro，对应另个一个agent sinks
a2.sources.r1.type = avro
a2.sources.r1.bind = hadoop102
a2.sources.r1.port = 4141

# Describe the sink
a2.sinks.k1.type = hdfs
a2.sinks.k1.hdfs.path = hdfs://hadoop102:9000/flume2/%Y%m%d/%H
#上传文件的前缀
a2.sinks.k1.hdfs.filePrefix = flume2-
#是否按照时间滚动文件夹
a2.sinks.k1.hdfs.round = true
#多少时间单位创建一个新的文件夹
a2.sinks.k1.hdfs.roundValue = 1
#重新定义时间单位
a2.sinks.k1.hdfs.roundUnit = hour
#是否使用本地时间戳
a2.sinks.k1.hdfs.useLocalTimeStamp = true
#积攒多少个Event才flush到HDFS一次
a2.sinks.k1.hdfs.batchSize = 100
#设置文件类型，可支持压缩
a2.sinks.k1.hdfs.fileType = DataStream
#多久生成一个新的文件
a2.sinks.k1.hdfs.rollInterval = 600
#设置每个文件的滚动大小大概是128M
a2.sinks.k1.hdfs.rollSize = 134217700
#文件的滚动与Event数量无关
a2.sinks.k1.hdfs.rollCount = 0
#最小冗余数
a2.sinks.k1.hdfs.minBlockReplicas = 1

# Describe the channel
a2.channels.c1.type = memory
a2.channels.c1.capacity = 1000
a2.channels.c1.transactionCapacity = 100

# Bind the source and sink to the channel
a2.sources.r1.channels = c1
a2.sinks.k1.channel = c1
```



