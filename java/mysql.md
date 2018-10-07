## mysql 5.7.23安装

* 配置文件

```shell
[client]
port = 3306
default-character-set=utf8mb4
socket=/tmp/mysql.sock

[mysql]
port = 3306
default-character-set=utf8mb4
socket=/tmp/mysql.sock

[mysqld]
##########################
# summary
##########################
bind-address = 0.0.0.0
port = 3306
basedir=/usr/local/mysql
datadir=/data/mysql/data
socket=/tmp/mysql.sock
tmpdir = /tmp
pid-file=/tmp/mysqld.pid
#skip-grant-tables
#skip-networking

explicit_defaults_for_timestamp=1
lower_case_table_names=1

table_open_cache = 8000

##########################
# time out
##########################
connect_timeout = 20
wait_timeout = 86400

##########################
# connection
##########################
max_connections = 2000
max_user_connections = 1900
max_connect_errors = 100000
max_allowed_packet = 1G

##########################
# character set
##########################
character-set-server = utf8mb4
collation-server = utf8mb4_bin

##########################
# log bin
##########################
server-id = 1
log_bin = mysql-bin
binlog_format = MIXED
sync_binlog = 1
expire_logs_days =7
binlog_cache_size = 128m
max_binlog_cache_size =512m
max_binlog_size =256M

binlog_ignore_db=mysql
binlog_ignore_db=information_schema
binlog_ignore_db=performation_schema
binlog_ignore_db=sys

##########################
# log relay
##########################
relay_log = mysql-relay-bin
relay_log_purge = on
relay_log_recovery = on
max_relay_log_size = 1G

##########################
# log error
##########################
log_error=/data/mysql/logs/error.log

##########################
# log slow
##########################
slow_query_log = on
slow_query_log_file = /data/mysql/logs/slow.log
long_query_time = 2
log_queries_not_using_indexes = on

##########################
# log general
##########################
general_log = on
general_log_file = /data/mysql/logs/gener.log


##########################
# thread pool
##########################
#thread_handling=pool-of-threads
#thread_handling=one-thread-per-connection
#thread_pool_oversubscribe=8 

##########################
# innodb
##########################
innodb_file_per_table=1
innodb_log_file_size=1024M
innodb_log_buffer_size=64M
```

*  创建软连接

```shell
ln -s mysql-5.7.23-linux-glibc2.12-x86_64/ mysq	
```

* 创建mysql用户和组

```shell
groupadd mysql
useradd mysql -g mysql
useradd -g mysql mysql
```

* 将mysql_home下载所有文件设置为mysql用户和组

```shell
chown -R mysql:mysql ./
```

* 初始化数据库

> mysql_install_db被废弃状态，使用mysqld --initialize，初始化时最后会生成root用户的默认密码 
>
> ./bin/mysqld --defaults-file=/etc/my.cnf --datadir=/data/mysql/data  --basedir=/usr/local/mysql/ --user=mysql --initialize 

```shell
 ./bin/mysqld --initialize --datadir=/mnt/mysql/data --user=mysql --basedir=/usr/local/mysql-5.7.23-linux-glibc2.12-x86_64
2018-09-26T01:43:29.045517Z 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
2018-09-26T01:43:30.111752Z 0 [Warning] InnoDB: New log files created, LSN=45790
2018-09-26T01:43:30.241817Z 0 [Warning] InnoDB: Creating foreign key constraint system tables.
2018-09-26T01:43:30.307720Z 0 [Warning] No existing UUID has been found, so we assume that this is the first time that this server has been started. Generating a new UUID: 921100ee-c12d-11e8-9749-00163e0cddb5.
2018-09-26T01:43:30.310006Z 0 [Warning] Gtid table is not ready to be used. Table 'mysql.gtid_executed' cannot be opened.
2018-09-26T01:43:30.310587Z 1 [Note] A temporary password is generated for root@localhost: otE/;h%CR47B

```

* 配置mysql启动文件

> 原有系统中如果存在/etc/my.cnf，把该文件删除掉

```shell
cp support-files/mysql.server /etc/init.d/mysqld
chmod 755 /etc/init.d/	
```

* 设置启动文件拥有执行权限

```shell
chmod 755 /etc/init.d/mysqld
```

* 连接mysql后需要先修改密码

> use mysql 通过update方式已经不支持了，默认密码初始化数据库时已经生成了

```shell
alter user user() identified by "root";
```



## mysql 5.7.23多实例安装

* 一台机器部署两个mysql实例进程分别对应款口号3306、3307

### 初始化数据库

```shell
./mysqld    --defaults-file=/data/mysql3306/my3306.cnf   --user=mysql --basedir=/usr/local/mysql  --initialize-insecure 
```

### 启动实例

```shell
./mysqld_safe --defaults-file=/data/mysql3306/my3306.cnf &
```

### 设置密码

```shell
mysqladmin -uroot -p password 123456 -S /data/mysql3306/mysql3306.sock
```

### 登陆

```shell
mysql -uroot -p -S /data/mysql3306/mysql3306.sock
```

### 关闭实例

```shell
# 多实例关闭
./mysqladmin -uroot -p -S /data/mysql3306/mysql3306.sock shutdown
# 单实例关闭
./mysqladmin -uroot -proot shutdown
```

## 编码

```sql
show variables like 'char%';
```

配置文件中在[client]、[mysqld]、[mysql]节点中配置字符集

## mysql文件

* log-bin：二进制日志文件

* log-error：记录严重的警告信息和错误信息，每次启动和关闭的详细信息，默认关闭

* slow-query-log-file：记录查询的sql语句，默认关闭

* 数据文件：

  ​	frm：表结构

  ​	myd：数据文件

  ​	myi：索引文件

## 备份

```shell
 mysqldump  -uzyb -pzyb123456 canteen > canteen_backup.sql
 mysqldump  -uroot -proot mysql > ~/mysql.sql
 mysql -hhostname -uusername -ppassword databasename < ‘backupfile
```

