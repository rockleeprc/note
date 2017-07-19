
## 核心组件生命周期

### SqlSessionFactoryBuilder

通过XML或者Java编码获得资源，构建SqlSessionFactory（可以构建多个），作用就是一个构建器，生命周期存在于方法局部，生成SqlSessionFactory对象后丢弃

### SqlSessionFactory

创建SqlSession，生命周期存在于整个MyBatis应用中，每个数据库只对应一个SqlSessionFactory，避免Connection消耗

### SqlSession

相当于JDBC的Connection，线程不安全，生命周期在于请求数据库处理事物的过程中

### Mapper

一个interface没有实现类，MyBatis根据这个接口生成代理对象，代理对象根据接口全路径+方法名去匹配xml文件中的sql，生命周期在一个SqlSession事物内

## 参数传递
* Map:影响可读性

* @Param:参数<5时最佳选择

* JavaBean:多参数时最佳选择

## MyBatis核心组件

* SqlSession：作为MyBatis工作的主要顶层API，表示和数据库交互的会话，完成必要数据库增删改查功能；

* Executor：MyBatis执行器，是MyBatis 调度的核心，负责SQL语句的生成和查询缓存的维护；

* StatementHandler：封装了JDBC Statement操作，负责对JDBC statement 的操作，如设置参数、将Statement结果集转换成List集合。

* ParameterHandler：负责对用户传递的参数转换成JDBC Statement 所需要的参数；

* ResultSetHandler：负责将JDBC返回的ResultSet结果集对象转换成List类型的集合；

* TypeHandler：负责java数据类型和jdbc数据类型之间的映射和转换；

* SqlSource：负责根据用户传递的parameterObject，动态地生成SQL语句，将信息封装到BoundSql对象中，并返回；

* BoundSql：表示动态生成的SQL语句以及相应的参数信息；

* Configuration：MyBatis所有的配置信息都维持在Configuration对象之中；

* MappedStatement：MappedStatement维护了一条<select|update|delete|insert>节点的封装；

## SqlSession源码

sqlSession.selectList("com.xxx.xxx.xxx.selectById",params);

	public class DefaultSqlSession implements SqlSession {
	  @Override
	  public <E> List<E> selectList(String statement, Object parameter) {
	    return this.selectList(statement, parameter, RowBounds.DEFAULT);
	  }

	  @Override
	  public <E> List<E> selectList(String statement, Object parameter, RowBounds rowBounds) {
	    try {
				// 通过com.xxx.xxx.xxx.selectById在Configuration中查找MappedStatement
	      MappedStatement ms = configuration.getMappedStatement(statement);
				// 将MappedStatement委托给Executor执行器执行
	      return executor.query(ms, wrapCollection(parameter), rowBounds, Executor.NO_RESULT_HANDLER);
	    } catch (Exception e) {
	      throw ExceptionFactory.wrapException("Error querying database.  Cause: " + e, e);
	    } finally {
	      ErrorContext.instance().reset();
	    }
	  }
	｝

xxMapper.xml配置文件信息会被维护成一个MappedStatement对象，保存在Configuration中的一个Map中

## Executor源码

	public abstract class BaseExecutor implements Executor {
	  @Override
	  public <E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler) throws SQLException {
			// 1. 根据具体传入的参数，动态地生成需要执行的SQL语句，用BoundSql对象表示  
	    BoundSql boundSql = ms.getBoundSql(parameter);
			// 2. 为当前的查询创建一个缓存Key
	    CacheKey key = createCacheKey(ms, parameter, rowBounds, boundSql);
	    return query(ms, parameter, rowBounds, resultHandler, key, boundSql);
	 }

	  @SuppressWarnings("unchecked")
	  @Override
	  public <E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql) throws SQLException {
	    ErrorContext.instance().resource(ms.getResource()).activity("executing a query").object(ms.getId());
	    if (closed) {
	      throw new ExecutorException("Executor was closed.");
	    }
	    if (queryStack == 0 && ms.isFlushCacheRequired()) {
	      clearLocalCache();
	    }
	    List<E> list;
	    try {
	      queryStack++;
	      list = resultHandler == null ? (List<E>) localCache.getObject(key) : null;
	      if (list != null) {
					// 3. 缓存中获取  
	        handleLocallyCachedOutputParameters(ms, key, parameter, boundSql);
	      } else {
					// 3. 缓存中没有值，直接从数据库中读取数据  
	        list = queryFromDatabase(ms, parameter, rowBounds, resultHandler, key, boundSql);
	      }
	    } finally {
	      queryStack--;
	    }
	    if (queryStack == 0) {
	      for (DeferredLoad deferredLoad : deferredLoads) {
	        deferredLoad.load();
	      }
	      // issue #601
	      deferredLoads.clear();
	      if (configuration.getLocalCacheScope() == LocalCacheScope.STATEMENT) {
	        // issue #482
	        clearLocalCache();
	      }
	    }
	    return list;
	  }

	  private <E> List<E> queryFromDatabase(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql) throws SQLException {
	    List<E> list;
	    localCache.putObject(key, EXECUTION_PLACEHOLDER);
	    try {
				//4. 执行查询，返回List 结果，然后    将查询的结果放入缓存之中
	      list = doQuery(ms, parameter, rowBounds, resultHandler, boundSql);
	    } finally {
	      localCache.removeObject(key);
	    }
	    localCache.putObject(key, list);
	    if (ms.getStatementType() == StatementType.CALLABLE) {
	      localOutputParameterCache.putObject(key, parameter);
	    }
	    return list;
	  }

	｝

	public class SimpleExecutor extends BaseExecutor {
	  @Override
	  public <E> List<E> doQuery(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) throws SQLException {
	    Statement stmt = null;
	    try {
	      Configuration configuration = ms.getConfiguration();
				//5. 根据既有的参数，创建StatementHandler对象来执行查询操作
	      StatementHandler handler = configuration.newStatementHandler(wrapper, ms, parameter, rowBounds, resultHandler, boundSql);
				//6. 创建java.Sql.Statement对象，传递给StatementHandler对象  
	      stmt = prepareStatement(handler, ms.getStatementLog());
				//7. 调用StatementHandler.query()方法，返回List结果集
	      return handler.<E>query(stmt, resultHandler);
	    } finally {
	      closeStatement(stmt);
	    }
	  }

	  private Statement prepareStatement(StatementHandler handler, Log statementLog) throws SQLException {
	    Statement stmt;
	    Connection connection = getConnection(statementLog);
			// 对创建的Statement对象设置参数，即设置SQL 语句中 ? 设置为指定的参数
	    stmt = handler.prepare(connection, transaction.getTimeout());
	    handler.parameterize(stmt);
	    return stmt;
	  }
	｝

### BaseExecutor/SimpleExecutor功能

1. 根据传递的参数，完成SQL语句的动态解析，生成BoundSql对象，供StatementHandler使用；
2. 为查询创建缓存，以提高性能；
3. 创建JDBC的Statement连接对象，传递给StatementHandler对象，返回List查询结果；

## StatementHandler

	public class PreparedStatementHandler extends BaseStatementHandler {
	  @Override
	  public void parameterize(Statement statement) throws SQLException {
 			//使用DefaultParameterHandler对象来完成对Statement的设值    
	    parameterHandler.setParameters((PreparedStatement) statement);
	  }

		@Override
		public <E> List<E> query(Statement statement, ResultHandler resultHandler) throws SQLException {
			// 1.调用preparedStatemnt.execute()方法，然后将resultSet交给ResultSetHandler处理   
			PreparedStatement ps = (PreparedStatement) statement;
			ps.execute();
			// 2.使用ResultHandler来处理ResultSet
			return resultSetHandler.<E> handleResultSets(ps);
		}
	}

	public class DefaultParameterHandler implements ParameterHandler {
	  @Override
	  public void setParameters(PreparedStatement ps) {
	    ErrorContext.instance().activity("setting parameters").object(mappedStatement.getParameterMap().getId());
	    List<ParameterMapping> parameterMappings = boundSql.getParameterMappings();
	    if (parameterMappings != null) {
	      for (int i = 0; i < parameterMappings.size(); i++) {
	        ParameterMapping parameterMapping = parameterMappings.get(i);
	        if (parameterMapping.getMode() != ParameterMode.OUT) {
	          Object value;
	          String propertyName = parameterMapping.getProperty();
	          if (boundSql.hasAdditionalParameter(propertyName)) { // issue #448 ask first for additional params
	            value = boundSql.getAdditionalParameter(propertyName);
	          } else if (parameterObject == null) {
	            value = null;
	          } else if (typeHandlerRegistry.hasTypeHandler(parameterObject.getClass())) {
	            value = parameterObject;
	          } else {
	            MetaObject metaObject = configuration.newMetaObject(parameterObject);
	            value = metaObject.getValue(propertyName);
	          }
						// 每一个Mapping都有一个TypeHandler，根据TypeHandler来对preparedStatement进行设置参数  
	          TypeHandler typeHandler = parameterMapping.getTypeHandler();
	          JdbcType jdbcType = parameterMapping.getJdbcType();
	          if (value == null && jdbcType == null) {
	            jdbcType = configuration.getJdbcTypeForNull();
	          }
	          try {
							// 设置参数
	            typeHandler.setParameter(ps, i + 1, value, jdbcType);
	          } catch (TypeException e) {
	            throw new TypeException("Could not set parameters for mapping: " + parameterMapping + ". Cause: " + e, e);
	          } catch (SQLException e) {
	            throw new TypeException("Could not set parameters for mapping: " + parameterMapping + ". Cause: " + e, e);
	          }
	        }
	      }
	    }
	  }
	}

## ResultSetHandler

	public class DefaultResultSetHandler implements ResultSetHandler {
		@Override
	  public List<Object> handleResultSets(Statement stmt) throws SQLException {
	    ErrorContext.instance().activity("handling results").object(mappedStatement.getId());

	    final List<Object> multipleResults = new ArrayList<Object>();

	    int resultSetCount = 0;
	    ResultSetWrapper rsw = getFirstResultSet(stmt);

	    List<ResultMap> resultMaps = mappedStatement.getResultMaps();
	    int resultMapCount = resultMaps.size();
	    validateResultMapsCount(rsw, resultMapCount);
	    while (rsw != null && resultMapCount > resultSetCount) {
	      ResultMap resultMap = resultMaps.get(resultSetCount);
				//将resultSet
	      handleResultSet(rsw, resultMap, multipleResults, null);
	      rsw = getNextResultSet(stmt);
	      cleanUpAfterHandlingResultSet();
	      resultSetCount++;
	    }

	    String[] resultSets = mappedStatement.getResultSets();
	    if (resultSets != null) {
	      while (rsw != null && resultSetCount < resultSets.length) {
	        ResultMapping parentMapping = nextResultMaps.get(resultSets[resultSetCount]);
	        if (parentMapping != null) {
	          String nestedResultMapId = parentMapping.getNestedResultMapId();
	          ResultMap resultMap = configuration.getResultMap(nestedResultMapId);
	          handleResultSet(rsw, resultMap, null, parentMapping);
	        }
	        rsw = getNextResultSet(stmt);
	        cleanUpAfterHandlingResultSet();
	        resultSetCount++;
	      }
	    }

	    return collapseSingleResultList(multipleResults);
	  }
	}
