## 概述

### kafka术语

1. broker：一台kafka服务器

2. 消息格式：

   1. key：对消息做partion时使用，消息保存在某个topic的哪个partition
   2. value：消息实际数据
   3. timestamp：消息发送时间戳，用于流式处理和依赖时间的处理语义，不指定时取当前时间

3. topic：逻辑概念上代表一类消息，可以使用topic区分实际业务

4. replica：针对partition的副本，kafka保证一个partition的多个replica不会分配到同一个broker上

5. topic-partitions-message（生产者offset/消费者offset）

   1. topic
      1. partition0
         1. message(offset)
         2. message(offset)
      2. partition1
         1. message(offset)
         2. message(offset)

6. ISR：kafka动态维护了一个leader的partition replica集合，当replica中的数据落后于leader时将被提出isr，当数据和leader同步时被加入到isr中



### 消息模型

* 消息队列
  1. 消息队列（queue）
  2. 发送者（sender）
  3. 接收者（receiver）
  4. p2p点对点消息传递
* 发布/订阅
  1. 主题（topic）：逻辑语义相近的消息容器
  2. 发布者（publisher）：将消息发送到topic
  3. 订阅者（subscriber）：从topic中获取信息

### 概要设计

#### 高吞吐量、低延迟

1. 批处理方式处理消息
2. 大量使用操作系统页缓存，内存操作命中率高，基本不需要磁盘读取
3. 采用数据追加方式，不使用磁盘随机读取操作
4. 不直接参与物理I/O，使用操作系统sendfile为代表的零拷贝技术

#### 持久化方式

* 数据立刻写入文件系统的持久化日志，通知客户端消息写入成功，减少内存消耗

#### 负载均衡和故障转移





## 安装

## producer开发

### producer工作流程

1. producer使用主线程将待发送消息封装到ProducerRecord类实例
2. 将ProducerRecord实例序列化后发送给partitioner
3. partitioner确定分区后发送到producer中的一块缓冲区
4. producer另一个工作线程（sender线程）实时从缓冲区中提取数据封装到一个批次（batch）
5. 由batch统一发送给对应的broker

### 异常

#### 可重试异常

1. LeaderNotAvaliableException：leader换届选举
2. NotControllerException：controller经历选举
3. NetworkException：网络故障

## 命令

```shell
# 创建topic
kafka-topics --create --zookeeper temporaryProd3:2181,temporaryProd4:2181,temporaryProd5:2181 --topic kafka2 --partitions 3 --replication-factor 3
# 生产者
kafka-console-producer --broker-list node4:9092 --topic test
# 消费者
kafka-console-consumer --bootstrap-server temporaryProd1:9092,temporaryProd2:9092,temporaryProd3:9092, --topic canal_syscenter_hj --from-beginning
# 查看topic
kafka-topics --zookeeper temporaryProd3:2181,temporaryProd4:2181,temporaryProd5:2181 --list
# 查看topic描述信息
kafka-topics --describe --zookeeper node1:2181,node2:2181,node3:218	1 --topic kafka2
# 删除topic
kafka-topics --delete --zookeeper node1:2181,node2:2181,node3:2181 --topic canteen
# producer吞吐量
kafka-producer-perf-test --topic test-topic --num-records 500000 --record-size 200 --throughput -1 --producer-props bootstrap.servers=node3:9092,node4:9092,node5:9092 acks=-1
# consumer吞吐量
kafka-consumer-perf-test --broker-list node3:9092,node4:9092,node5:9092 --fetch-size 200 --messages 500000 --topic test-topic
# 查看偏移量
kafka-run-class kafka.tools.GetOffsetShell --broker-list node4:9092 --topic canteen --time -1

# 查看偏移量
kafka-consumer-groups --bootstrap-server prod1:9092,prod2:9092,prod3:9092,prod4:9092,prod5:9092,prod6:9092,prod7:9092  --describe --group g9
# 重置偏移量
kafka-consumer-groups  --bootstrap-server prod1:9092,prod2:9092,prod3:9092,prod4:9092,prod5:9092,prod6:9092,prod7:9092 --group g9 --topic  cg9  --execute --reset-offsets --to-offset 274180742
```

