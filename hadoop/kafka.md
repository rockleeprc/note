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

1. 大量使用操作系统页缓存，内存操作命中率高，基本不需要磁盘读取
2. 采用数据追加方式，不使用磁盘随机读取操作
3. 不直接参与物理I/O，使用操作系统sendfile为代表的零拷贝技术

#### 持久化方式

* 数据立刻写入文件系统的持久化日志，通知客户端消息写入成功，减少内存消耗

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

