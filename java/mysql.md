



## mysql 5.7.23安装

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





```shell
 mysqldump  -uzyb -pzyb123456 canteen > canteen_backup.sql
 mysqldump  -uroot -proot mysql > ~/mysql.sql
 mysql -hhostname -uusername -ppassword databasename < ‘backupfile
```

