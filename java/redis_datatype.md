

## string

* 字符串可以存储3种类型
	* 字符串，byte string
	* 整数，取值范围和系统的长整数的取之范围相同（32位系统，32位有符号整数）
	* 浮点数，IEEE745标准的双精度浮点数


### set/get/setex/mset/mget

	# 插入
	127.0.0.1:6379> set s1 "zhangsan"
	OK
	# 读取
	127.0.0.1:6379> get s1
	"zhangsan"
	# 指定存活时间为10s
	127.0.0.1:6379> setex s2 10 "lisi"
	OK
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

### incr/incrby/decr/decrby

	# 对值+1
	127.0.0.1:6379> set key 1
	OK
	127.0.0.1:6379> get key
	"4"
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
	# 
	127.0.0.1:6379> incrby k4 10
	127.0.0.1:6379> decrby k4 7


## list

* 对链表数据结构的支持，可以有序的存储多个字符串
* 列表的主要优点在于可以包含多个字符串值

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



## hash

## set

## sortset
