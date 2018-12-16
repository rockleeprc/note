## 基础数据

* 表

  ```mysql
  create table person(
  	id bigint(20) not null auto_increment,
  	name varchar(20) null,
  	age int null,
  	primary key(id)
  );
  ```

* 数据

  ```mysql
  insert into person(name) values ('张三丰');
  insert into person(name) values ('李逵');
  insert into person(name) values ('关羽');
  insert into person(name,age) values ('曹操',18);
  insert into person(name) values ('张三丰');
  insert into person(name) values ('李逵');
  insert into person(name) values ('关羽');
  insert into person(name) values ('潘金');
  insert into person(age) values(99);
  insert into person(name,age) values(null,null);
  ```

## 导入

* 查询数据库，测试连通性

  ```shell
  sqoop list-databases --connect jdbc:mysql://172.18.123.139:3306/ --username root --password root
  ```

* 增量更新

  ```shell
  sqoop job \
  --create jobperson \
  -- import \
  --connect jdbc:mysql://172.18.123.139/test?useSSL=false \
  --username root \
  --password root \
  --password-file hdfs-path \
  --table person \
  --last-value 1 \
  --check-column  id \
  --incremental  append \
  --target-dir /mysql/person \
  --num-mappers 1 \
  --fields-terminated-by "\t"
  ```

* 连接mysql输入每次都是需要输入密码

  ```shell
  # sqoop规定密码文件必须放在HDFS之上，并且权限必须为400
  （1）echo -n "123456" > sqoopMysqlTest.pwd
  （2）hdfs dfs -put sqoopMysqlTest.pwd /data/cdh/hive/hiveExternal
  （3）hdfs dfs -chmod 400 /data/cdh/hive/hiveExternal/sqoopMysqlTest.pwd
  ```

* job相关命令

  ```shell
  sqoop job --list　　　　　　　　　   列出所有的job
  sqoop job --show jobname　　　　显示jobname的信息
  sqoop job --delete jobname 　　　删除jobname
  sqoop job --exec  jobname  　　　执行jobname
  ```

## 导出

* 增量导出

  ```shell
  sqoop export \
  --connect jdbc:mysql://172.18.123.139/test?characterEncoding=utf-8&useSSL=false \
  --username root \
  --password root \
  --table person \
  --update-mode allowinsert \
  --update-key id \
  --num-mappers 1 \
  --export-dir /mysql/person/ \
  --input-fields-terminated-by "\t" \
  --columns="id,name,age" 
  ```
