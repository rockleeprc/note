## explain格式

```sql
mysql> explain select * from user_info where id=2 \G;
*************************** 1. row ***************************
           id: 1	# SELECT 查询的标识符. 每个 SELECT 都会自动分配一个唯一的标识符.
  select_type: SIMPLE # SELECT 查询的类型.
        table: user_info # 查询的是哪个表
   partitions: NULL # 查询的是哪个表
         type: const # join 类型
possible_keys: PRIMARY # 此次查询中可能选用的索引
          key: PRIMARY # 此次查询中确切使用到的索引.
      key_len: 8 
          ref: const # 哪个字段或常数与 key 一起被使用
         rows: 1 	# 显示此查询一共扫描了多少行. 这个是一个估计值.
     filtered: 100.00	#  表示此查询条件所过滤的数据的百分比
        Extra: NULL	# 额外的信息
1 row in set, 1 warning (0.00 sec)

```

## id

* select查询的一个序列号，包含一组数字
  * id相同表示mysql内部的查询优化器执行命令，也就是加载表的顺序的，从上到下，当然也有可能顺序不是这样 
  * 如果是子查询，id的序号会递增，id值越大优先级越高，越先被执行
  * id相同存在，不同也存在，先执行id最大的，相同id的从上到下

## select_type

- SIMPLE, 表示此查询不包含 UNION 查询或子查询
- PRIMARY, 表示此查询是最外层的查询
- UNION, 表示此查询是 UNION 的第二或随后的查询
- DEPENDENT UNION, UNION 中的第二个或后面的查询语句, 取决于外面的查询
- UNION RESULT, UNION 的结果
- SUBQUERY, 子查询中的第一个 SELECT
- DEPENDENT SUBQUERY: 子查询中的第一个 SELECT, 取决于外面的查询. 即子查询依赖于外层查询的结果. 

## table

* 表示查询涉及的表或衍生表

## type

### system

* 表中只有一条数据. 这个类型是特殊的 `const` 类型

### const

* 针对主键或唯一索引的等值查询扫描, 最多只返回一行数据，const 查询速度非常快, 因为它仅仅读取一次即可，例如下面的这个查询, 它使用了主键索引, 因此 `type` 就是 `const` 类型的

### eq_ref

* 此类型通常出现在多表的 join 查询,  表示对于前表的每一个结果, 都只能匹配到后表的一行结果. 并且查询的比较操作通常是 `=`, 查询效率较高 

### ref

* 此类型通常出现在多表的 join 查询, 针对于非唯一或非主键索引, 或者是使用了 `最左前缀` 规则索引的查询.  

### range

* 表示使用索引范围查询, 通过索引字段范围获取表中部分数据记录. 这个类型通常出现在 =, <>, >, >=, <, <=, IS NULL, <=>, BETWEEN, IN() 操作中. 当 `type` 是 `range` 时, 那么 EXPLAIN 输出的 `ref` 字段为 NULL, 并且 `key_len` 字段是此次查询中使用到的索引的最长的那个. 

### index

* 表示全索引扫描(full index scan), 和 ALL 类型类似, 只不过 ALL 类型是全表扫描, 而 index 类型则仅仅扫描所有的索引, 而不扫描数据. `index` 类型通常出现在: 所要查询的数据直接在索引树中就可以获取到, 而不需要扫描数据. 当是这种情况时, Extra 字段 会显示 `Using index` 

### all

* 表示全表扫描, 这个类型的查询是性能最差的查询之一. 通常来说, 我们的查询不应该出现 ALL 类型的查询, 因为这样的查询在数据量大的情况下, 对数据库的性能是巨大的灾难. 如一个查询是 ALL 类型查询, 那么一般来说可以对相应的字段添加索引来避免. 

### type性能排序

* ALL 
* index 
* range ~ index_merge 
* ref
* eq_ref 
* const 
* system