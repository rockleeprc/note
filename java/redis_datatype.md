

## 全局命令

### keys/dbsize/exists/ttl/type/object encoding
	# 查看所有key，遍历利所有key
	127.0.0.1:6379> keys *
	1) "sort"
	# 查看键总数，不会遍历所有key
	127.0.0.1:6379> dbsize
	(integer) 2
	# key是否存在
	127.0.0.1:6379> exists list
	(integer) 1
	# 不存在返回0
	127.0.0.1:6379> exists not_exist_key
	(integer) 0
	# 设置过期时间
	127.0.0.1:6379> expire hello 10
	(integer) 1
	# 剩余过期时间
	127.0.0.1:6379> ttl hello
	(integer) 7
	# key的数据类型
	127.0.0.1:6379> type list
	list
	# 查看内部编码
	127.0.0.1:6379> object encoding hello
	"embstr"
	127.0.0.1:6379> object encoding count
	"int"


## string

* 字符串内部编码
	* int：8字节长整型
	* embstr：小于等于39字节的字符串
	* raw：大于39个字节的字符串

* 字符串可以存储3种类型
	* 字符串，byte string
	* 整数，取值范围和系统的长整数的取之范围相同（32位系统，32位有符号整数，64位系统...）
	* 浮点数，IEEE745标准的双精度浮点数
* 不能超过512MB

### set/get/mset/mget

	# 插入
	127.0.0.1:6379> set s1 "zhangsan"
	OK
	# 读取
	127.0.0.1:6379> get s1
	"zhangsan"
	# 插入多条数据
	127.0.0.1:6379> mset s2 "lisi" s3 "wangwu"
	OK
	# 读取多条数据
	127.0.0.1:6379> mget s1 s2 s3
	1) "zhangsan"
	2) "lisi"
	3) "wangwu"
	# 删除操作
	127.0.0.1:6379> del s2 s1
	(integer) 2

### setex/setnx/setxx
	# 指定存活时间为10s
	127.0.0.1:6379> setex s2 10 "lisi"
	OK
	# 不存在设置成功，用于添加
	127.0.0.1:6379> setnx hello world
	(integer) 1
	127.0.0.1:6379> setnx hello world
	(integer) 0
	# 存在设置成功，用于更新
	127.0.0.1:6379> set hello world xx
	OK
	127.0.0.1:6379> set hello1 world xx
	(nil)


### incr/incrby/decr/decrby

	# 一个不存在key或空串，将键值当作0处理
	127.0.0.1:6379> incr k2
	(integer) 1
	127.0.0.1:6379> decr k4
	(integer) -1
	# 对无法被解释为整数或浮点数的值进行+1/-1将返回一个错误
	127.0.0.1:6379> set k1 v
	OK
	127.0.0.1:6379> incr k1
	(error) ERR value is not an integer or out of range
	# 设置每次加/减的值
	127.0.0.1:6379> incrby k4 10
	127.0.0.1:6379> decrby k4 7

## list

* 对链表数据结构的支持，可以有序的存储多个字符串值

### rpush/lpush/rpop/lpop/lindex/lrange/ltrim

	# 右插入
	127.0.0.1:6379> rpush k 1 2 3
	(integer) 3
	127.0.0.1:6379> rpush k 4 5 6
	(integer) 6
	# 左插入
	127.0.0.1:6379> lpush k 0 -1 -2 -3
	(integer) 10
	# 获取list中的数据，从0开始，-1表示全部
	127.0.0.1:6379> lrange k 0 -1
	 1) "-3"
	 2) "-2"
	 3) "-1"
	 4) "0"
	 5) "1"
	 6) "2"
	 7) "3"
	 8) "4"
	 9) "5"
	10) "6"
	# 获取指定位置的数据
	127.0.0.1:6379> lindex k 3
	"0"

## set
* list、set都可存储字符串，list可以存储相同的字符串，set保证存储的每个字符串唯一
* redis以无序的方式来存储多个各不相同的元素

### sadd/smembers/sismember/srem
	# 增加元素
	127.0.0.1:6379> sadd setkey i1 i2 i3 i1 i2
	(integer) 3
	# 获取元素
	127.0.0.1:6379> smembers setkey
	1) "i3"
	2) "i1"
	3) "i2"
	# 判断i1是否在key=setkey中
	127.0.0.1:6379> sismember setkey i1
	(integer) 1
	# 删除setkey中i1、i2元素
	127.0.0.1:6379> srem setkey i1 i2
	(integer) 2

## hash
* 可以将多个key-value对存储在一个redis key里面，值可以是字符串和数值，并可以对数值执行增/自减

### hset/hget/hgetall/hdel
	# 增加，成功返回1
	127.0.0.1:6379> hset hash k1 v1
	(integer) 1
	# 失败返回0
	127.0.0.1:6379> hset hash k1 v1
	(integer) 0
	# 通过key获取元素
	127.0.0.1:6379> hget hash k1
	"v1"
	# 获取素有key-value
	127.0.0.1:6379> hgetall hash
	1) "k1"
	2) "v1"
	3) "k2"
	4) "v2"

### hmset/hmget/hlen
	# 批量增加key-value
	127.0.0.1:6379> hmset hash k3 v3 k4 v4
	OK
	# 批量获取
	127.0.0.1:6379> hmget hash k1 k2 k4
	1) "v1"
	2) "v2"
	3) "v4"
	# 获取key-value数量
	127.0.0.1:6379> hlen hash
	(integer) 4

### hexists/hkeys/hvals
	# key是否存在，存在返回1，不存在返回0
	127.0.0.1:6379> hexists hash k1
	(integer) 1
	# 返回所有的key
	127.0.0.1:6379> hkeys hash
	# 返回所有的value
	127.0.0.1:6379> hvals hash



## sort set
* 有序的集合，key成为member，value称为score，唯一一个可以根据memeber和score访问元素，score只能按照排序顺序访问

### zadd/zrem/zrange/zrangebyscore
	# 添加 score member
	127.0.0.1:6379> zadd zset 1 m1 2 m2 3 m3
	(integer) 3
	# 获取
	127.0.0.1:6379> zrange zset 0 -1
	1) "m1"
	2) "m2"
	3) "m3"
	# 通过member获取
	127.0.0.1:6379> zrangebyscore zset 1 2
	1) "m1"
	2) "m2"
	# 删除
	127.0.0.1:6379> zrem zset m1 m2
	(integer) 2

	127.0.0.1:6379> zadd sort 10 a 30 b 20 c 
	(integer) 3
	# 按照score排名，b，c，a
	127.0.0.1:6379> zrevrank sort a
	(integer) 2
	127.0.0.1:6379> zrevrank sort b
	(integer) 0
	# 排名0-2
	127.0.0.1:6379> zrevrange sort 0 2
	1) "b"
	2) "c"
	3) "a"
