
## redis单机

### redis编译
在源码所在目录使用make命令对redis的源码工程进行编译

### redis安装

	[root@hdp01 redis-3.2.4]#make PREFIX=/usr/local/redis install

### 修改配置文件
从源码包中copy一份redis.conf文件到安装目录，修改配置参数为后台进程
	
	daemonize yes

### 启动redis
启动redis并指定使用配置文件

	[root@hdp01 redis3.2]# bin/redis-server redis.conf
	
	#查看redis的是否启动
	[root@hdp01 redis3.2]# ps -ef | grep redis
	root 2131 1 0 21:42 ? 00:00:00 bin/redis-server 127.0.0.1:6379
	root 2138 2106 0 21:42 pts/1 00:00:00 bin/redis-cli
	root 2169 2067 0 21:57 pts/0 00:00:00 grep redis
	[root@hdp01 redis3.2]# netstat -nltp | grep redis
	tcp 0 0 127.0.0.1:6379 0.0.0.0:* LISTEN 2131/bin/redis-serv 

### 关闭redis

	[root@hdp01 redis3.2]# bin/redis-cli shutdown
	
	[root@hdp01 redis3.2]# bin/redis-cli -a jedis shutdown

### 连接redis
	[root@hdp01 redis3.2]# bin/redis-cli
	127.0.0.1:6379> ping
	PONG
	127.0.0.1:6379>

## redis持久化

* rdb（默认）：在指定的时间间隔写入硬盘
	* 优势，只有一个文件，方便压缩转移
	* 缺点，如果宕机，没有持久化的数据将丢失，数据损失较大
* aof：以日志的方式记录每一个操作的命令，服务器启动后就构建数据库
	* 优势，安全性相对rdb方式高很多
	* 缺点，效率相对rdb方式低很多

### rdb
redis.conf 默认配置

	# Save the DB on disk:
	# 每900秒内至少有1个key发生变化，就持久化
	save 900 1	
	# 每300秒内至少有10个key发生变化，就持久化
	save 300 10
	# 每60秒内至少有10000个key发生变化，就持久化
	save 60 10000
	# 保存的rdb文件名称
	dbfilename dump.rdb
	# 保存的路径，默认当前路径
	dir ./

### aop

	# 默认关闭
	appendonly no
	# 记录日志的文件
	appendfilename "appendonly.aof"
	
	# 只要发生修改立即同步，安全性高
	# appendfsync always
	# 每秒同步一次
	appendfsync everysec
	# 不同步
	# appendfsync no

### 数据恢复

数据恢复时只要把*.rdb/*.aof文件copy到dir配置的位置，启动redis就可以





