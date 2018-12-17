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

### 排序

#### 数据准备

```mysql
# dept
create external table if not exists test.dept(
    deptno int,
    dname string,
    loc int
)
row format delimited fields terminated by '\t';

10	ACCOUNTING	1700
20	RESEARCH	1800
30	SALES	1900
40	OPERATIONS	1700

# emp
create external table if not exists test.emp(
    empno int,
    ename string,
    job string,
    mgr int,
    hiredate string, 
    sal double, 
    comm double,
    deptno int)
row format delimited fields terminated by '\t';

7369	SMITH	CLERK	7902	1980-12-17	800.00		20
7499	ALLEN	SALESMAN	7698	1981-2-20	1600.00	300.00	30
7521	WARD	SALESMAN	7698	1981-2-22	1250.00	500.00	30
7566	JONES	MANAGER	7839	1981-4-2	2975.00		20
7654	MARTIN	SALESMAN	7698	1981-9-28	1250.00	1400.00	30
7698	BLAKE	MANAGER	7839	1981-5-1	2850.00		30
7782	CLARK	MANAGER	7839	1981-6-9	2450.00		10
7788	SCOTT	ANALYST	7566	1987-4-19	3000.00		20
7839	KING	PRESIDENT		1981-11-17	5000.00		10
7844	TURNER	SALESMAN	7698	1981-9-8	1500.00	0.00	30
7876	ADAMS	CLERK	7788	1987-5-23	1100.00		20
7900	JAMES	CLERK	7698	1981-12-3	950.00		30
7902	FORD	ANALYST	7566	1981-12-3	3000.00		20
7934	MILLER	CLERK	7782	1982-1-23	1300.00		10

# location
create table if not exists test.location(
    loc int,
    loc_name string
)
row format delimited fields terminated by '\t';

1700	Beijing
1800	London
1900	Tokyo
```

#### order by

* 全局排序，一个reduce，默认asc

#### sort by

* 每个reduce内部排序，最终结果不是排序的

  ```mysql
  # 查看reduce数量
  > set mapreduce.job.reduces;
  # 设置reduce数量
  > set mapreduce.job.reduces=3;
  # 将查询结果导入到文件中，每个文件中的数据按部门编号排序，生成的文件数是reduce个数
  > insert overwrite local directory '/opt/module/datas/sortby-result'
   select * from emp sort by deptno desc;
  ```

#### distribute by

* 类似MR中partition，进行分区，结合sort by使用，Hive要求DISTRIBUTE BY语句要写在SORT BY语句之前

  ```mysql
  > set mapreduce.job.reduces=3;
  # 先按照部门编号分区，再按照员工编号降序排序
  > insert overwrite local directory '/opt/module/datas/distribute-result' select * from emp distribute by deptno sort by empno desc;
  ```

#### cluster by

* 当distribute by和sorts by字段相同时，可以使用cluster by方式

* cluster by除了具有distribute by的功能外还兼具sort by的功能。但是排序只能是升序排序，不能指定排序规则为ASC或者DESC

  ```mysql
  > select * from emp cluster by deptno;
  > select * from emp distribute by deptno sort by deptno;
  ```

### 查询函数

#### NVL( string1, replace_with)

```mysql
# 如果员工的comm为NULL，则用-1代替
> select nvl(comm,-1) from emp;
```

#### CASE WHEN

```mysql
create table emp_sex(
    name string, 
    dept_id string, 
    sex string) 
row format delimited fields terminated by "\t";
load data local inpath '/opt/module/datas/emp_sex.txt' into table emp_sex;

悟空	A	男
大海	A	男
宋宋	B	男
凤姐	A	女
婷姐	B	女
婷婷	B	女

# 求出不同部门男女各多少人
select 
  dept_id,
  sum(case sex when '男' then 1 else 0 end) male_count,
  sum(case sex when '女' then 1 else 0 end) female_count
from 
  emp_sex
group by
  dept_id;
```

#### 行转列

* CONCAT(string A/col, string B/col…)：返回输入字符串连接后的结果，支持任意个输入字符串;
* CONCAT_WS(separator, str1, str2,...)：它是一个特殊形式的 CONCAT()。第一个参数剩余参数间的分隔符。分隔符可以是与剩余参数一样的字符串。如果分隔符是 NULL，返回值也将为 NULL。这个函数会跳过分隔符参数后的任何 NULL 和空字符串。分隔符将被加到被连接的字符串之间;
* COLLECT_SET(col)：函数只接受基本数据类型，它的主要作用是将某字段的值进行去重汇总，产生array类型字段。

```mysql
create table person_info(
    name string, 
    constellation string, 
    blood_type string) 
row format delimited fields terminated by "\t";
load data local inpath "/opt/module/datas/person_info.txt" into table person_info;

孙悟空	白羊座	A
大海	     射手座	A
宋宋	     白羊座	B
猪八戒	白羊座	A
凤姐	     射手座	A

# 把星座和血型一样的人归类到一起
select
    t1.base,
    concat_ws('|', collect_set(t1.name)) name
from
    (select
     name,
     concat(constellation, ",", blood_type) base
     from
     person_info) t1
group by
    t1.base;
```

#### 列转行

* EXPLODE(col)：将hive一列中复杂的array或者map结构拆分成多行。
* LATERAL VIEW用法：LATERAL VIEW udtf(expression) tableAlias AS columnAlias，解释：用于和split, explode等UDTF一起使用，它能够将一列数据拆成多行数据，在此基础上可以对拆分后的数据进行聚合。

```mysql
create table movie_info(
    movie string, 
    category array<string>) 
row format delimited fields terminated by "\t"
collection items terminated by ",";
load data local inpath "/opt/module/datas/movie.txt" into table movie_info;

《疑犯追踪》	悬疑,动作,科幻,剧情
《Lie to me》	悬疑,警匪,动作,心理,剧情
《战狼2》	战争,动作,灾难

# 将电影分类中的数组数据展开
select
    movie,
    category_name
from 
    movie_info lateral view explode(category) table_tmp as category_name;
```

#### 窗口函数

* OVER()：指定分析函数工作的数据窗口大小，这个数据窗口大小可能会随着行的变而变化
* CURRENT ROW：当前行
* n PRECEDING：往前n行数据
* n FOLLOWING：往后n行数据
* UNBOUNDED：起点，UNBOUNDED PRECEDING 表示从前面的起点， UNBOUNDED FOLLOWING表示到后面的终点
* LAG(col,n)：往前第n行数据
* LEAD(col,n)：往后第n行数据
* NTILE(n)：把有序分区中的行分发到指定数据的组中，各个组有编号，编号从1开始，对于每一行，NTILE返回此行所属的组的编号。注意：n必须为int类型。

```mysql
create table business(
    name string, 
    orderdate string,
    cost int
) ROW FORMAT DELIMITED FIELDS TERMINATED BY ',';
load data local inpath "/opt/module/datas/business.txt" into table business;

jack,2017-01-01,10
tony,2017-01-02,15
jack,2017-02-03,23
tony,2017-01-04,29
jack,2017-01-05,46
jack,2017-04-06,42
tony,2017-01-07,50
jack,2017-01-08,55
mart,2017-04-08,62
mart,2017-04-09,68
neil,2017-05-10,12
mart,2017-04-11,75
neil,2017-06-12,80
mart,2017-04-13,94

# 查询在2017年4月份购买过的顾客及总人数
select name,count(*) over () 
from business 
where substring(orderdate,1,7) = '2017-04' 
group by name;

# 查询顾客的购买明细及月购买总额
select name,orderdate,cost,sum(cost) over(partition by month(orderdate)) 
from business;

# 要将cost按照日期进行累加
select name,orderdate,cost, 
-- 所有行相加 
sum(cost) over() as sample1,
-- 按name分组，组内数据相加 
sum(cost) over(partition by name) as sample2,
-- 按name分组，组内数据累加 
sum(cost) over(partition by name order by orderdate) as sample3,
-- 和sample3一样,由起点到当前行的聚合 
sum(cost) over(partition by name order by orderdate rows between UNBOUNDED PRECEDING and current row ) as sample4,
-- 当前行和前面一行做聚合 
sum(cost) over(partition by name order by orderdate rows between 1 PRECEDING and current row) as sample5, 
-- 当前行和前边一行及后面一行 
sum(cost) over(partition by name order by orderdate rows between 1 PRECEDING AND 1 FOLLOWING ) as sample6,
-- 当前行及后面所有行 
sum(cost) over(partition by name order by orderdate rows between current row and UNBOUNDED FOLLOWING ) as sample7 
from business;

# 查看顾客上次的购买时间
select name,orderdate,cost, 
lag(orderdate,1,'1900-01-01') over(partition by name order by orderdate ) as time1, lag(orderdate,2) over (partition by name order by orderdate) as time2 
from business;

# 查询前20%时间的订单信息
select * from (
    select name,orderdate,cost, ntile(5) over(order by orderdate) sorted
    from business
) t
where sorted = 1;
```


> 聚合函数：**对于每个组只返回一行**

> 分析函数：**对于每个组返回多行**

- 基础结构

  ```mysql
  # 分析函数（如:sum(),max(),row_number()...） + 窗口子句（over函数）
  # 先根据cookieid字段分区，相同的cookieid分为一区，每个分区内根据createtime字段排序（默认升序）
  # 不加 partition by 的话则把整个数据集当作一个分区
  # 不加 order by的话会对某些函数统计结果产生影响，如sum()
  over（partition by cookieid order by createtime）
  
  -- 关键是理解 ROWS BETWEEN 含义,也叫做window子句： 
  -- PRECEDING：往前 
  -- FOLLOWING：往后 
  -- CURRENT ROW：当前行 
  -- UNBOUNDED：无边界
  	-- UNBOUNDED PRECEDING 表示从最前面的起点开始
  	-- UNBOUNDED FOLLOWING：表示到最后面的终点 
  -- 其他AVG，MIN，MAX，和SUM用法一样
  
  SELECT cookieid,createtime,pv,
  -- 默认为从起点到当前行
  SUM(pv) OVER(PARTITION BY cookieid ORDER BY createtime) AS pv1, 
  -- 从起点到当前行，结果同pv1 
  SUM(pv) OVER(PARTITION BY cookieid ORDER BY createtime ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS pv2, 
  -- 当前行+往前3行
  SUM(pv) OVER(PARTITION BY cookieid ORDER BY createtime ROWS BETWEEN 3 PRECEDING AND CURRENT ROW) AS pv3,
  -- 当前行+往前3行+往后1行   
  SUM(pv) OVER(PARTITION BY cookieid ORDER BY createtime ROWS BETWEEN 3 PRECEDING AND 1 FOLLOWING) AS pv4,    
  -- 当前行+往后所有行  
  SUM(pv) OVER(PARTITION BY cookieid ORDER BY createtime ROWS BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING) AS pv5   
  FROM test1;
  # 首先 PARTITION BY cookieid，根据cookieid分区，各分区之间默认根据字典顺序排序
  # ORDER BY createtime，指定的是分区内部的排序，默认为升序
  
  # 查询结果，这些窗口的划分都是在分区内部！超过分区大小就无效了
  # 如果不指定ROWS BETWEEN,默认统计窗口为从起点到当前行
  # 如果不指定ORDER BY，则将分组内所有值累加
  cookieid    createtime  pv  pv1  pv2    pv3  pv4  pv5
  a           2017-12-01  3    3    3      3    3    3
  b           2017-12-00  3    3    3      3    3    3
  cookie1     2017-12-10  1    1    1      1    6    26
  cookie1     2017-12-11  5    6    6      6    13   25
  cookie1     2017-12-12  7    13  13      13   16   20
  cookie1     2017-12-13  3    16  16      16   18   13
  cookie1     2017-12-14  2    18  18      17   21   10
  cookie1     2017-12-15  4    22  22      16   20   8
  cookie1     2017-12-16  4    26  26      13   13   4
  cookie2     2017-12-12  7    7    7      7    13   14
  cookie2     2017-12-16  6    13  13      13   14   7
  cookie2     2017-12-24  1    14  14      14   14   1
  cookie3     2017-12-22  5    5    5      5     5   5
  
  # 如果没有order by，不仅分区内没有排序，sum()计算的pv也是整个分区的pv
  # max()函数无论有没有order by 都是计算整个分区的最大值
  select cookieid,createtime,pv,sum(pv) over(PARTITION BY cookieid) as pv1 
  FROM test1
  ```

* 窗口函数总结
  * partiion by：等同于group by
  * 窗口函数和聚合函数的不同：
    * sum()函数可以根据每一行的窗口返回各自行对应的值，有多少行记录就有多少个sum值，
    * group by只能计算每一组的sum，每组只有一个值
    * sum()计算的是分区内排序后一个个叠加的值，和order by有关

#### NTILE 函数

* NTILE(n)，用于将分组数据按照顺序切分成n片，返回当前切片值

  * 注1：如果切片不均匀，默认增加第一个切片的分布
  * 注2：NTILE不支持ROWS BETWEEN

  ```mysql
  SELECT cookieid,createtime,pv,
  -- 分组内将数据分成2片
  NTILE(2) OVER(PARTITION BY cookieid ORDER BY createtime) AS ntile1, 
  -- 分组内将数据分成3片
  NTILE(3) OVER(PARTITION BY cookieid ORDER BY createtime) AS ntile2,  
  -- 将所有数据分成4片
  NTILE(4) OVER(PARTITION BY cookieid ORDER BY createtime) AS ntile3   
  FROM test1 
  
  # 用法举例： 统计一个cookie，pv数最多的前1/3的天，取 ntile = 1 的记录，就是我们想要的结果
  SELECT cookieid,createtime,pv,
  NTILE(3) OVER(PARTITION BY cookieid ORDER BY pv DESC) AS ntile 
  FROM test1;
  ```

#### ROW_NUMBER 函数

* ROW_NUMBER() 从1开始，按照顺序，生成分组内记录的序列

  ```mysql
  SELECT cookieid,createtime,pv,
  ROW_NUMBER() OVER(PARTITION BY cookieid ORDER BY pv desc) AS rn  
  FROM test1;
  ```

  > ROW_NUMBER() 的应用场景非常多，比如获取分组内排序第一的记录、获取一个session中的第一条refer等


#### Rank/DENSE_RANK

```mysql
create table score(
    name string,
    subject string, 
    score int) 
row format delimited fields terminated by "\t";
load data local inpath '/opt/module/datas/score.txt' into table score;

孙悟空	语文	87
孙悟空	数学	95
孙悟空	英语	68
大海	语文	94
大海	数学	56
大海	英语	84
宋宋	语文	64
宋宋	数学	86
宋宋	英语	84
婷婷	语文	65
婷婷	数学	85
婷婷	英语	78


select name,
subject,
score,
rank() over(partition by subject order by score desc) rp,
dense_rank() over(partition by subject order by score desc) drp,
row_number() over(partition by subject order by score desc) rmp
from score;
```

* `RANK()` 生成数据项在分组中的排名，**排名相等会在名次中留下空位** 

* `DENSE_RANK()` 生成数据项在分组中的排名，**排名相等会在名次中不会留下空位**

  ```mysql
  # 把 rank、dense_rank、row_number三者对比，这样比较清晰
  SELECT cookieid,createtime,pv,
  RANK() OVER(PARTITION BY cookieid ORDER BY pv desc) AS rank1,
  DENSE_RANK() OVER(PARTITION BY cookieid ORDER BY pv desc) AS d_rank2,
  ROW_NUMBER() OVER(PARTITION BY cookieid ORDER BY pv DESC) AS rn3 
  FROM test1 
  ```
