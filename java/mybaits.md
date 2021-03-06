## 映射文件

### CRUD

* <select>
  * resultType
    * 返回一个pojo对象
    * 返回一个pojo对象的集合
    * 返回一个map
      * @MapKey()
  * resultMap
* <insert>
  * parameterType
  * useGeneratedKeys
  * keyProperty
  * <selectKey>：oracle中查询主键值
    * keyProperty：查询的主键值映射到哪个javabean属性中
    * resultType：生成主键的返回值
    * order：在插入isnert语句之前运行，“BEFORE”
* <update>
* <delete>

> cud可以在mapper接口中返回boolean，当返回的记录数>0时为true

### 参数处理

* mapper接口中只传一个参数时，xml文件中的#{}中可以是任何名称，因为只有一个参数
* mapper接口中多个参数时，被封装为map，map的key为param1、param2…，value是参数值
  * @Param：明确指定map中的key的值
  * 传递pojo对象
  * 传递map对象
  * 传递集合时，Collection key为collection，List key 为list，数组key为array
* ${}：取出值，直接拼装到sql语句中
  * 分表查询时，表名无法使用占位符
  * order by时，字段无法使用占位符，asc、desc 无法使用占位符
* #{}：预编译形式，使用占位符，只能获取参数，对于表名，排序字段无效
  * jdbcType：
    * jdbcTypeForNull：全局配置，java null字段映射为数据库的NULL，默认对应数据为OTHER

TODO 参数源码

### 查询
* 关联查询返回值
  * <resultMap>
  * <association>：关联查询、分步查询、延迟加载
    * select：分段查询的sql
    * column：映射列
  * <collection>：一对多时，查询一的一方，关联查询多的一方
    * ofType：集合内元素类型
    * select：
    * column：
    * fetchType：是否延迟加载，默认lazy，可以配置eager马上加载
  * <discriminator>
    * case

## 动态sql

## 缓存

### 一级缓存

* 与数据库同一次回话中，获取相同数据时

* sqlSession级别的缓存，永远开启，无法关闭
* 一级缓存失效
  * 不是同一个sqlSession
  * 两次查询sql条件不同
  * 两个查询间执行了insert、update、delete
    * 这些标签中flushCache=true是默认配置，同时清除一级和二级缓存
    * select中flushCache=false
  * 手动调用clearCache()清除一级缓存

### 二级缓存

* 基于namespace级别的缓存，一个namespace对应一个二级缓存
* 如果sqlSession关闭，sqlSession中的数据会被保存到二级缓存中
* 配置
  * cacheEnabled，mybaits配置文件中全局配置，默认开启
  * <cache>，在mapper.xml中配置使用二级缓存
  * <select>中的useCache可以关闭二级缓存
* 自定义缓存策略实现Cache接口
* pojo需要实现Serializable
* localCacheScope，设置为STATEMENT时，可以禁用一级缓存，3.3中的新功能

### ehcache

* 在github mybatis项目中有ehcache-mybatis项目
* <cache>中type属性配置EhcacheCache
* 配置encache.xml

## mybatis-spring

### spring

* web.xml，contextConfigLocation，spring主配置文件
* web.xml，DispatcherServlet，springmvc前端控制器
  * 在DispatcherServlet配置中，配置contextConfigLocation，加载springmvc配置文件

### mybaits

* 在spring主配置文件中配置SqlSessionFactoryBean
  * dataSource，数据源，交给spring管理
  * configLoaciton，mybaits配置文件
  * mapperLocations，mapper.xml配置文件的位置
* <mybatis-spring:scan>，扫做所有mapp接口（替代MapperScannerConfigurer）

## mybatis generator



## mybaits工作原理

### mybatis层次结构

* 接口层
  * mapper接口
* 数据处理层
  * 参数映射，ParameterHandler
  * sql解析，SqlSource
  * sql执行，Executor
  * 结果处理和映射，ResultSetHandler
* 框架支撑层
  * xml配置
  * 注解
  * 事务管理
  * 缓存机制
  * 连接池管理
* 引导层
  * xml配置
  * java api

### SqlSessionFactoryBuilder

* XmlConfigBuilder，创建xml解析器
  * 解析mybatis配置文件
  * 解析mapper.xml文件
* Configuration，所有配置文件的详细信息
  * mybatis文件所有配置信息
  * mapper.xml文件所有配置信息
  * MappedStatement，每一个curd标签封装为MappedStatement对象
  * MapperRegistry
* DefaultSqlSessionFactory，包含配置信息

> 解析所有配置文件，将配置信息保存在Configuration中，创建SqlSessionFactory实例

### SqlSessionFacotry

* Executor

  * 根据Configuration中的ExecutorType创建(默认SimpleExectutor)

  * 如果开启二级缓存，用CachingExecutor包装一下

  * ###### 拦截器链包装Executor

* DefaultSqlSession

> 将Configuration、Executor封装到DefaultSqlSession中

### SqlSession

* MapperProxyFactory
  * 通过Configuration->MapperRegistry获得
  * 创建MapperProxy，实例化Mapper接口
* MapperProxy
  * InvocationHandler
  * 返回MapperProxy代理的Mapper接口实例

> Mapper的代理对象包含了MapperProxy、SqlSession、Mappe接口、Map实例作为一级缓存

### MapperProxy

* 将调用方法包装为MapperMethod实例
* 通过SqlSession进行具体的查询操作
  * 最终调用的都是selectList()
  * 通过id在Configuration中获取MappedStatement
  * 将MappedStatement传给Executor
* Executor（CachingExecutor）
  * BoundSql，与sql语句相关信息
  * 创建缓存key
  * 调用SimpleExecutor
  * 二级缓存->一级缓存->sql
  * BaseExecutor
    * doQuery()
    * 通过Configuration创建StatementHanlder
    * 构造器中创建ParameterHandler、ResultSetHandler
* StatementHandler
  * PrepareStatementHandler
  * 添加到拦截器链
  * 通过ParameterHandler设置参数
  * 调用原生JDBC
  * 通过ResultSetHandler设置返回结果集
* ParameterHandler
  * 创建DefaultParameterHandler
  * 调用TypeHandler设置参数
* ResultSetHandler
  * DefaultResultSetHandler
  * 通过TypeHandler获取结果集
* TypeHandler
  * 数据库类型和java类型的映射


## 插件

### 开发步骤

* 实现Interceptor接口
* @Intercepts插件签名
* 配置插件
=======
### Configuration

* 构造方法初始化时，用TypeAliasRegistry注册系统级别名(如：JDBC、POOLED、SLF4J)







- Executor（update、query、flushStatements、commint、rollback、getTransaction、close、isClosed）:执行器处理
- StatementHandler（prepare、parameterize、batch、update、query）：Sql语法构建处理 
- ParameterHandler（getParameterObject、setParameters）：参数 处理
- ResultSetHandler（handleResultSets、handleOutputParameters）：结果集处理

> 执行流程：SqlSession -> Executor -> StatementHandler -> ParameterHandler ->  ResultSetHandler 






