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

# spring连载：
## BeanFactoryPostProcessor
* 允许自定义钩子修改ApplicationContext中的BeanDefinition
* 对于使用配置文件中的值修改ApplicationContext中的配置bean非常拥有，PropertyResourceConfigurer实现采用BeanFactoryPostProcessor实现
* BeanFactoryPostProcessor可以操作BeanDefinition，但不能操作bean实例对象，在BeanFactoryPostProcessor中操作bean实例可能导致bean过早实例化，操作bean实例使用BeanPostProcessor
* ApplicationContext自动在其BeanDefinition中检测BeanFactoryPostProcessor bean，并在创建任何其他bean之前应用它们，BeanFactoryPostProcessor也可以用ConfigurableApplicationContext以编程方式注册
* 在ApplicationContext中的BeanFactoryPostProcessor可以根据org.springframework.core.PriorityOrdered和org.springframework.core.Ordered语义进行排序
* ConfigurableApplicationContext编程注册的BeanFactoryPostProcessorbean将按注册顺序应用
* BeanFactoryPostProcessor不考虑@Order注解
```java
@FunctionalInterface
public interface BeanFactoryPostProcessor {

	/**
     * 在标准初始化后修改ApplicationContext内部的bean factory，
     * 所有BeanDefinition定义都将被加载，但bean还没有被实例化，允许对beand重写或添加属性，甚至对bean过早实例化
	 */
	void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException;

}
```
## BeanPostProcessor 
* 允许自定义钩子修改bean的实例，比如：检查baen的标记接口或用代理包装bean实例
* ApplicationContext自动在BeanDefinition中自动检测到BeanPostProcessor bean，在ApplicationContext中的BeanPostProcessor可以根据org.springframework.core.PriorityOrdered和org.springframework.core.Ordered语义进行排序
* BeanFactory允许对后处理器进行编程注册，将它们应用于BeanFactory创建的所有bean，以编程方式向BeanFactory注册的BeanPostProcessorbean将按注册顺序应用，对于以编程方式注册的后处理器，通过实现PriorityOrdered或Ordered接口表达的任何排序语义都将被忽略
* BeanPostProcessorbean不考虑@Order注解
```java
public interface BeanPostProcessor {

	/**
    * 1、在任何bean初始化前回调，比如在InitializingBean的afterPropertiesSet()或自定义init-method之前
    * 2、bean中的属性已经被填充
    * 3、返回的bean实例可能是原始bean的包装类，默认原样返回
    */
	@Nullable
	default Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
		return bean;
	}

	/**
    * 1、在任何bean初始化后回调，比如InitializingBean的afterPropertiesSet()或自定义init-method之后
    * 2、返回的bean实例可能是原始bean的包装类，默认原样返回
    * 3、作为FactoryBean实例和FactoryBean创建对象实例的回调方法，后处理器可以通过相应的FactoryBean检查bean实例来决定是应用于FactoryBean还是应用于*    创建的对象，或者两者都应用
    * 4、InstantiationAwareBeanPostProcessor.postProcessBeforeInstantiation()后也将调用改方法
    */
	@Nullable
	default Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
		return bean;
	}
}
```


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
