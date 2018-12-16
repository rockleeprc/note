## Hive安装

### mysql安装

```mysql
# 修改密码
SET PASSWORD=PASSWORD('000000');
select User, Host, Password from user;
# 修改host主机，删除root用户其它host
update user set host='%' where host='localhost';
# 刷新权限
flush privileges;
```

### 配置修改

* 拷贝mysql驱动到hive/lib/下

#### hive-env.sh

```shell
export JAVA_HOME=/usr/apps/jdk1.8.0_181-amd64
# 配置hadoop环境变量
export HADOOP_HOME=/opt/module/hadoop-2.7.2
export HIVE_HOME=/home/hadoop/apps/hive
# 配置hive配置目录
export HIVE_CONF_DIR=/opt/module/hive/conf
```

#### hive-site.xml（自己创建）

```xml
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
	<!-- 配置mysql连接 -->
    <property>
        <name>javax.jdo.option.ConnectionURL</name>
        <value>jdbc:mysql://hadoop102:3306/metastore?createDatabaseIfNotExist=true</value>
        <description>JDBC connect string for a JDBC metastore</description>
    </property>
	<!-- 配置mysql Driver -->
    <property>
        <name>javax.jdo.option.ConnectionDriverName</name>
        <value>com.mysql.jdbc.Driver</value>
        <description>Driver class name for a JDBC metastore</description>
    </property>
	<!-- 配置mysql 用户名 -->
    <property>
        <name>javax.jdo.option.ConnectionUserName</name>
        <value>root</value>
        <description>username to use against metastore database</description>
    </property>
	<!-- 配置mysql 密码 -->
    <property>
        <name>javax.jdo.option.ConnectionPassword</name>
        <value>000000</value>
        <description>password to use against metastore database</description>
    </property>
    <!-- 配置hive默认仓库的位置 -->
    <property>
        <name>hive.metastore.warehouse.dir</name>
        <value>/user/hive/warehouse</value>
        <description>location of default database for the warehouse</description>
    </property>
    <!-- 配置查询时显示列信息 -->
    <property>
        <name>hive.cli.print.header</name>
        <value>true</value>
    </property>
    <!-- 配置查询时显示库名 -->
    <property>
        <name>hive.cli.print.current.db</name>
        <value>true</value>
    </property>
</configuration>
```

#### hive-log4j.properties

```properties
# 修改log日志目录
hive.log.dir=/opt/module/hive/logs
```

## Hive命令

### 常用交互

```shell
# 帮助
$ bin/hive -help
# 不进入hive执行sql
$ bin/hive -e "select id from student;"
# 执行脚本中sql
$ bin/hive -f /opt/module/datas/hivef.sql
$ bin/hive -f /opt/module/datas/hivef.sql  > /opt/module/datas/hive_result.txt
# 查看hdfs
hive(default)>dfs -ls /;
# 查看本地文件系统
hive(default)>! ls /opt/module/datas;
```

### DDL

* 创建数据库

  ```mysql
  > create database db_hive;
  > create database if not exists db_hive;
  # 创建仓库并指定数据在hdfs存放的位置
  > create database db_hive2 location '/db_hive2.db';
  # 创建数据库描述信息
  > alter database db_hive set dbproperties('createtime'='20170830');
  > desc database extended db_hive;
  db_name comment location        owner_name      owner_type      parameters
  db_hive         hdfs://hadoop102:8020/user/hive/warehouse/db_hive.db    atguigu USER    {createtime=20170830}
  ```

* 删除数据库

  ```mysql
  > drop database if exists db_hive2;
  ```

* 建表

  ```mysql
  # 创建外部表
  > create external table if not exists te.emp(
      empno int,
      ename string,
      job string,
      mgr int,
      hiredate string, 
      sal double, 
      comm double,
      deptno int)
  row format delimited fields terminated by '\t';
  ```

* 导入数据

  ```mysql
  # 本地文件系统
  > load data local inpath '/opt/module/datas/dept.txt' into table default.dept;
  # 省略local从hdfs查找
  > load data inpath '/opt/module/datas/dept.txt' into table default.dept;
  ```

* 查看表，并格式化数据

  ```mysql
  > desc formatted dept;
  ```

* 分区

  ```mysql
  # 创建分区表
  > create table dept_partition(
      deptno int, dname string, loc string
  )
  partitioned by (month string)
  row format delimited fields terminated by '\t';
  # 加载数据到分区中
  > load data local inpath '/opt/module/datas/dept.txt' into table default.dept_partition partition(month='201709');
  # 增加分区，多个分区用空格
  > alter table dept_partition add partition(month='201706') ;
  > alter table dept_partition add partition(month='201705') partition(month='201704');
  # 删除多分区，多个分区用逗号
  > alter table dept_partition drop partition (month='201704');
  > alter table dept_partition drop partition (month='201705'), partition (month='201706');
  ```

### DML

#### load

```mysql
# 本地文件导入，copy操作，原始文件存在
> load data local inpath '/opt/module/datas/student.txt' into table default.student;
# hdfs文件导入，move操作，原始文件不存在
> load data inpath '/user/atguigu/hive/student.txt' into table default.student;
# 导入时覆盖已有数据使用overwrite
> load data inpath '/user/atguigu/hive/student.txt' overwrite into table default.student;
```

#### insert

```mysql
> insert overwrite table student partition(month='201708')
select id, name from student where month='201709';
```

#### as select

```mysql
# 根据查询结果创建表
> create table if not exists student3
as select id, name from student;
```

#### location

```mysql
# 创建表时指定路径
> create table if not exists student5(
    id int, name string
)
row format delimited fields terminated by '\t'
location '/user/hive/warehouse/student5';
```

