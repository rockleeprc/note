



### 环境准备

```shell
# 安装ruby环境
$ yum -y install ruby
$ yum -y install rubygems
# 安装ruby的redis插件
$ gem install redis
```

### redis编译安装

```shell
# 下载
$ wget http://download.redis.io/releases/redis-4.0.14.tar.gz
# 编译安装
$ make PREFIX=/usr/local/redis-4.0.14 install
# 软连
$ ln -s /usr/local/redis-4.0.14 /usr/local/redis
# 创建cluster需要的目录，conf保存配置文件，data保存数据，logs保存日志
$ mkdir -p /usr/local/redis/{conf,data,logs}
```

### cluster安装

```shell
# 启动所有节点，创建集群
$ ./redis-trib.rb create --replicas 1 192.168.13.213:6379 192.168.13.213:6380 192.168.13.214:6379 192.168.13.214:6380 192.168.13.215:6379 192.168.13.215:6380
>>> Creating cluster
>>> Performing hash slots allocation on 6 nodes...
Using 3 masters:
192.168.13.213:6379
192.168.13.214:6379
192.168.13.215:6379
Adding replica 192.168.13.214:6380 to 192.168.13.213:6379
Adding replica 192.168.13.215:6380 to 192.168.13.214:6379
Adding replica 192.168.13.213:6380 to 192.168.13.215:6379
M: bc2ac9edeb73205cce02ea81b963f20e2a938637 192.168.13.213:6379
   slots:0-5460 (5461 slots) master
S: d35d97e88bb561ace53c8fe57ac952c3ceeeb1de 192.168.13.213:6380
   replicates 1024092cca267f271d4a99d702752706d5af4336
M: 09d3478473498b2665132c707d8b89d90e962d0e 192.168.13.214:6379
   slots:5461-10922 (5462 slots) master
S: fbd03eac9f7e57abc2c8d9162b1489393b8fb019 192.168.13.214:6380
   replicates bc2ac9edeb73205cce02ea81b963f20e2a938637
M: 1024092cca267f271d4a99d702752706d5af4336 192.168.13.215:6379
   slots:10923-16383 (5461 slots) master
S: 72c416492087966d328f1bb19cfeacf14023459e 192.168.13.215:6380
   replicates 09d3478473498b2665132c707d8b89d90e962d0e
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join.
>>> Performing Cluster Check (using node 192.168.13.213:6379)
M: bc2ac9edeb73205cce02ea81b963f20e2a938637 192.168.13.213:6379
   slots:0-5460 (5461 slots) master
   1 additional replica(s)
M: 09d3478473498b2665132c707d8b89d90e962d0e 192.168.13.214:6379
   slots:5461-10922 (5462 slots) master
   1 additional replica(s)
S: d35d97e88bb561ace53c8fe57ac952c3ceeeb1de 192.168.13.213:6380
   slots: (0 slots) slave
   replicates 1024092cca267f271d4a99d702752706d5af4336
M: 1024092cca267f271d4a99d702752706d5af4336 192.168.13.215:6379
   slots:10923-16383 (5461 slots) master
   1 additional replica(s)
S: fbd03eac9f7e57abc2c8d9162b1489393b8fb019 192.168.13.214:6380
   slots: (0 slots) slave
   replicates bc2ac9edeb73205cce02ea81b963f20e2a938637
S: 72c416492087966d328f1bb19cfeacf14023459e 192.168.13.215:6380
   slots: (0 slots) slave
   replicates 09d3478473498b2665132c707d8b89d90e962d0e
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.

# 集群完整性检测
$ ./redis-trib.rb check node4:6379
>>> Performing Cluster Check (using node node4:6379)
S: 09d3478473498b2665132c707d8b89d90e962d0e node4:6379
   slots: (0 slots) slave
   replicates 72c416492087966d328f1bb19cfeacf14023459e
M: d35d97e88bb561ace53c8fe57ac952c3ceeeb1de 192.168.13.213:6380
   slots:10923-16383 (5461 slots) master
   1 additional replica(s)
M: bc2ac9edeb73205cce02ea81b963f20e2a938637 192.168.13.213:6379
   slots:0-5460 (5461 slots) master
   1 additional replica(s)
S: fbd03eac9f7e57abc2c8d9162b1489393b8fb019 192.168.13.214:6380
   slots: (0 slots) slave
   replicates bc2ac9edeb73205cce02ea81b963f20e2a938637
M: 72c416492087966d328f1bb19cfeacf14023459e 192.168.13.215:6380
   slots:5461-10922 (5462 slots) master
   1 additional replica(s)
S: 1024092cca267f271d4a99d702752706d5af4336 192.168.13.215:6379
   slots: (0 slots) slave
   replicates d35d97e88bb561ace53c8fe57ac952c3ceeeb1de
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.

# 检查节点之间槽的均衡性
$ ./redis-trib.rb rebalance node4:6379
>>> Performing Cluster Check (using node node4:6379)
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
*** No rebalancing needed! All nodes are within the 2.0% threshold.


```

### 配置文件

```shell
# 绑定端口
port 6379
# 绑定IP
bind 192.168.13.214
# 指定数据存放路径
dir /usr/local/redis/data/6379
# 启动集群模式
cluster-enabled yes
# 生成的集群配置文件名称，执行redis-trib.rb后自动生成
cluster-config-file /usr/local/redis/conf/nodes-6379.conf
# 后台启动
daemonize yes
# 指定集群节点超时时间
cluster-node-timeout 5000
# 指定持久化方式
appendonly yes
# pid文件 
pidfile /var/run/redis_6379.pid
# 日志
logfile "/usr/local/redis/logs/redis_6379.log"
# 是否所有节点都可用才算正常
cluster-require-full-coverage no
# 关闭rdb，主从同步默认开启rdb，无法关闭
save ""
```

### 集群信息

```shell
# 集群节点信息
> cluster nodes
# <id> <ip:port@cport> <flags> <master> <ping-sent> <pong-recv> <config-epoch> <link-state> <slot> <slot> ... <slot>
bc2ac9edeb73205cce02ea81b963f20e2a938637 192.168.13.213:6379@16379 master - 0 1572415198047 28 connected 0-5460
> cluster info
# 计算key落在那个slot里
> cluster keyslot key 
# slots点信息
> cluster slots
```

### 集群压测（只支持单节点）

```shell
# 100个并发连接，100000个请求，检测host端口为6379的redis服务器性能
$ redis-benchmark -h node3 -p 6379 -c 100 -n 1000000
# 测试存取大小为100字节的数据包的性能
$ redis-benchmark -h node3 -p 6379 -q -d 100  
# 测试set，lpush性能
$ redis-benchmark -h node3 -p 6379 -t set,lpush -n 100000 -q
# 计算key的slot
> cluster keyslot 20
```

### 集群扩容

```shell
# redis-trib add-node 要加的节点和端口  现有任意节点和端口
$ redis-trib add-node 192.168.0.201:7007 192.168.0.201:7001 
# 新加上来没有数据-及没有槽位，重新分片
$ redis-trib reshard 192.168.0.201:7007
```

### JedisCluster API

```java
JedisPoolConfig config = new JedisPoolConfig();
config.setMaxTotal(30);// 最大连接 默认8
config.setMaxIdle(10);// 空闲连接 默认8
config.setMinIdle(0);  //最小空闲连接数 默认0
config.setMaxWaitMillis(5000);// 获取连接时的最大等待毫秒数(如果设置为阻塞时BlockWhenExhausted),如果超时就抛异常, 小于零:阻塞不确定的时间,  默认-1
config.setTestOnBorrow(true);// 对拿到的connection进行validateObject校验

Set<HostAndPort> redisNodes = new HashSet<>();
redisNodes.addAll(getHostByDev());
/*
connectTimeOut 连接超时
soTimeOut 读写超时
maxAttemts 重试次数
*/
JedisCluster jedisCluster = new JedisCluster(redisNodes, 10000, 6000, 7, config);
```

 ### Tips

* 配置文件中bind 配置不能是127.0.0.1，执行redis-trib.rb脚本是会`Waiting for the cluster to join......`

* 重新执行redis-trib.rb时需要删除每个节点的数据目录和集群自动生成的conf文件

* `redis-trib.rb`在`redis_home/src`下

* 如果redis配置了密码认证，需要修改ruby文件的密码配置项`/usr/local/rvm/gems/ruby-2.4.1/gems/redis-4.1.2/lib/redis/client.rb`

* 目前的client JedisCluster是无法感知的，只能通过执行命令后, 服务端返回的“-MOVED”信息感知节点的变化，并以此来刷新缓存信息

  

  