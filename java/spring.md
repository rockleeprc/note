# 数据源路由

```java
// 保存了key和数据库连接的映射关系
private Map<Object, Object> targetDataSources;
// 默认的连接
private Object defaultTargetDataSource;
// 通过targetDataSources构建而来，存储结构也是数据库标识和数据源的映射关系
private Map<Object, DataSource> resolvedDataSources;
```

```java
// 设置路由的数据源集合
public void setTargetDataSources(Map<Object, Object> targetDataSources)
// 设置默认的数据源
public void setDefaultTargetDataSource(Object defaultTargetDataSource)
// 当前数据源对应的key
protected abstract Object determineCurrentLookupKey()
```

```java
public Connection getConnection() throws SQLException {
    return determineTargetDataSource().getConnection();
}
protected DataSource determineTargetDataSource() {
    Assert.notNull(this.resolvedDataSources, "DataSource router not initialized");
    Object lookupKey = determineCurrentLookupKey(); // 子类实现获取数据源key的规则
    DataSource dataSource = this.resolvedDataSources.get(lookupKey); // 没有直接使用targetDataSources
    if (dataSource == null && (this.lenientFallback || lookupKey == null)) {
        dataSource = this.resolvedDefaultDataSource;
    }
    if (dataSource == null) {
        throw new IllegalStateException("Cannot determine target DataSource for lookup key [" + lookupKey + "]");
    }
    return dataSource;
}
```

```java
@Override
public void afterPropertiesSet() {
    if (this.targetDataSources == null) {
        throw new IllegalArgumentException("Property 'targetDataSources' is required");
    }
    // 将targetDataSources转换为resolvedDataSources
    this.resolvedDataSources = new HashMap<>(this.targetDataSources.size());
    this.targetDataSources.forEach((key, value) -> {
        Object lookupKey = resolveSpecifiedLookupKey(key);
        DataSource dataSource = resolveSpecifiedDataSource(value);
        this.resolvedDataSources.put(lookupKey, dataSource);
    });
    // 将defaultTargetDataSource转换为resolvedDefaultDataSource
    if (this.defaultTargetDataSource != null) {
        this.resolvedDefaultDataSource = resolveSpecifiedDataSource(this.defaultTargetDataSource);
    }
}
```
* InitializingBean：bean实例已经创建好，且属性值和依赖的其他bean实例都已经注入以后执行
* 执行afterPropertiesSet()时，targetDataSources，defaultTargetDataSource必须已经初始化完成

# 扩展点
## BeanPostProcessor 
* postProcessBeforeInitialization()：在bean实例化前调用
* postProcessAfterInitialization()：在bean实例化后调用，如果bean实现了InitializingBean，则在执行完，该接口的afterPropertiesSet方法后调用 ，如果实现了init-method则，在执行完init-method后调用
* 调用点：
    调用类AbstractAutowireCapableBeanFactory.initializeBean(String beanName, Object bean, @Nullable RootBeanDefinition mbd)
        * invokeAwareMethods() 调用实现Aware接口方法
        * applyBeanPostProcessorsBeforeInitialization()  实现BeanPostProcessor类 的 postProcessBeforeInitialization方法
        * invokeInitMethods() 如果bean实现了InitializingBean 接口，则调用afterPropertiesSet方法，如果有配置init-method,则调用配置的初始化方法
        * applyBeanPostProcessorsAfterInitialization() 实现BeanPostProcessor类 的 postProcessAfterInitialization方法
## InitializingBean/init-method
* 为实现该接口的bean提供默认的初始化方法，也可以在xml配置bean的使用init-method来实现初始化方法
第一：实现InitializingBean接口，继而实现afterPropertiesSet的方法
第二：反射原理，配置文件使用init-method标签直接注入bean
* 调用点
    调用类AbstractAutowireCapableBeanFactory.invokeInitMethods(String beanName, Object bean, @Nullable RootBeanDefinition mbd)
* 应用场景
    * 在SpringMVC中AbstractHandlerMethodMapping就实现了InitializingBean接口，当一个RequestMappingHandlerMapping的实例创建完成后会接着调用afterPropertiesSet方法，扫描所有的controller完成所有的handler method的注册。
    * TransactionTemplate，判断transactionManager是否初始化，没有初始化抛异常

* 执行过程：构造方法-->BeanPostProcessor-->InitializingBean-->bean中的初始化方法
