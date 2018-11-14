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

## 索引优化

### 索引分类

- 单值索引：一个索引只包含一列，一个表可以多个单列索引
- 唯一索引：索引列的值必须唯一，但允许有空值
- 复合索引：一个索引包含多个列

### 创建索引

```
# idx_user_name：索引名	user：表名	name：字段名 
create unique|index idx_user_name on user(name)
drop index idx_user_name on user
```

* sql语句执行顺序

  from

  on

  where

  group by

  having

  select

  distinct

  order by

  limit

* 关联查询

  inner join：左右相同的部分

  left join：

  right join：

  左表独有：select * from a left join b on a.id = b.id where b.id is null

  右表独有：select * from a left join b on a.id = b.id where a.id is null

  全部（mysql不支持）：select from a full outer join b on a.id = b.id

  ​	select * from a left join b on a.id = b.id

  ​	union（自带去重功能）

  ​	select * from a left join b on a.id = b.id 

  左右独有：select * from a full outer join b on a.id=b.id where a.id is null or b.id is null 

* 索引：帮助mysql高效获取数据的数据结构（排序+快速查找）

## Explain

### id

* 查询序号，包含一组数字，表示查询中执行select子句或操作表的顺序
* id相同，执行顺序由上至下
* id不同，id值大的优先执行
* id相同/不同，先执行id值大的，相同id值的由上至下

### select_type

* simple，简单select查询，不包含子查询或union
* primary，包含子查询，最外层查询被标记为primary
* subquery，select或where中包含子查询
* derived，from中包含的子查询被标记为derived，结果放在临时表中
* union，第二个select出现在union后，被标记为union
* union result，从union表中获取结果的select

### table

* 本行数据是关于哪张表

### type（重点）

- system，表只有一条记录，const类型特例
- const，通过索引一次就能找到，const用于比较primary key或unique索引，在where条件中，msyql可以将查询转换为一个常量
- eq_ref，唯一索引扫描，表中只有一条记录与之匹配，常见于主键或唯一索引扫描
- ref，非唯一性索引扫描，通过索引访问，但返回匹配某个单独值的所有行
- range，检索指定的范围，使用一个索引选择数据，不扫描全部索引，只扫描部分，一般存在于between、<、>、in查询中
- index，全索引扫描

* all，全表扫描

> 最好 sytem>const>eq_ref>ref>range>index>all 最差
>
> 至少达到range，最好达到ref级别

### possible_keys

* 可能使用到的索引，一个或多个，但不一定被查询实际使用

### key（重点）

* 实际使用到的索引，null表示没有使用到索引
* 索引覆盖时，possible_keys为null

### key_len

* 索引中使用的字节数，长度越短越好

### ref

* 显示索引的哪个列被使用，如果可能最好是const

### rows

* 根据表的统计信息及索引选用情况，大致估算出找到所需的记录数

### Extra（重点）

* Using filesort，无法按照索引顺序读取，mysql使用外部索引排序
* Using temporary，使用临时表保存中间结果，常见于order by和group by
* Using index，select操作使用了覆盖索引，同时出现Using where表明索引被用来执行索引键值的查找，如果没有出现Using where表明索引用来读取数据而非执行查找动作（覆盖索引：select查询的列只从索引中获取，不必读取数据行）
* Using where
* Using join buffer，使用了连接缓存
* ipossible where，where子句的表达式返回false，不能用来检索任何数据
* select tables optimized away，在没有group by的情况下，基于索引min、max操作

## 备份

```shell
 mysqldump  -uzyb -pzyb123456 canteen > canteen_backup.sql
 mysqldump  -uroot -proot mysql > ~/mysql.sql
 mysql -hhostname -uusername -ppassword databasename < ‘backupfile
```

