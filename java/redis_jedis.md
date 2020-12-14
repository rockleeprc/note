

### jedis连接

```java
Jedis jedis = new Jedis("192.168.33.11", 6379);
String result = jedis.ping();
```

使用版本为redis-3.2.4.tar.gz，会出现一个连接拒绝的异常

```java
redis.clients.jedis.exceptions.JedisConnectionException:
 java.net.ConnectException: Connection refused: connect
Caused by: java.net.ConnectException: Connection refused: connect
```

在redis.conf中注释掉绑定127.0.0.1，这样Java客户端就可以连接，不然只能本地连接

```shell
bind 127.0.0.1
```

再次尝试连接，会有一个因为redis自身的保护模式没关出现的异常

```java
redis.clients.jedis.exceptions.JedisDataException:
DENIED Redis is running in protected mode because protected mode is enabled,
no bind address was specified, no authentication password is requested to clients.
In this mode connections are only accepted from the loopback interface.
If you want to connect from external computers to Redis you may adopt one of the following solutions:
1) Just disable protected mode sending the command 'CONFIG SET protected-mode no' from
the loopback interface by connecting to Redis from the same host the server is running,
however MAKE SURE Redis is not publicly accessible from internet if you do so.
Use CONFIG REWRITE to make this change permanent.
2) Alternatively you can just disable the protected mode by editing the Redis configuration file,
and setting the protected mode option to 'no', and then restarting the server.
3) If you started the server manually just for testing, restart it with the '--protected-mode no' option.
4) Setup a bind address or an authentication password. NOTE: You only need to do one of the above
things in order for the server to start accepting connections from the outside.
```

第一种解决方式：在redis.conf配置关闭保护模式，就可以正常PING/PONG

```shell
protected-mode no
```

第二种解决方式：设置认证

```java
# jedis为密码
127.0.0.1:6379> config set requirepass jedis
OK
# cli登录后认证
127.0.0.1:6379> auth jedis
OK
# 设置认证
Jedis jedis = new Jedis("192.168.33.11", 6379);
jedis.auth("jedis");
jedis.ping();
```

在redis.conf中也可以持久化认证密码，配置

```shell
requirepass jedis
```

### jedis连接池

```java
JedisPoolConfig config = new JedisPoolConfig();
config.setMaxTotal(100);//最大连接数
config.setMaxIdle(50);//最大空闲连接数

JedisPool jedisPool = new JedisPool(config, "192.168.33.11",6379);
Jedis jedis = jedisPool.getResource();
String ping = jedis.ping();
System.out.println(ping);
```


### 修改配置
```
config set maxmemory 20480mb
CONFIG SET SAVE "900 10 300 100 60 10000"
CONFIG REWRITE
```

### 优化
* 问题
```shell
333:M 16 Nov 14:00:27.100 * 10 changes in 300 seconds. Saving...
333:M 16 Nov 14:00:27.100 # Can't save in background: fork: Cannot allocate memory
```
* 解决
```
https://blog.csdn.net/zqz_zqz/article/details/53384854
https://www.cnblogs.com/godfather007/p/10167849.html
```