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

# spring连载：扩展点
## Aware 接口

## BeanFactoryPostProcessor
* 允许自定义hook修改ApplicationContext中的BeanDefinition
* 对于使用配置文件中的值修改ApplicationContext中的配置bean的属性非常拥有，PropertyResourceConfigurer实现采用BeanFactoryPostProcessor实现，还有PropertyPlaceholderConfigurer、PropertySourcesPlaceholderConfigurer将配置文件中的值注入到容器中的bean
* BeanFactoryPostProcessor可以操作BeanDefinition，但不能操作bean实例对象，在BeanFactoryPostProcessor中操作bean实例可能导致bean过早实例化，操作bean实例使用推荐使用BeanPostProcessor
* ApplicationContext自动在其BeanDefinition中检测BeanFactoryPostProcessor bean，并在创建任何其他bean之前调用它们
* 在ApplicationContext中的BeanFactoryPostProcessor可以根据org.springframework.core.PriorityOrdered和org.springframework.core.Ordered语义进行排序
* BeanFactoryPostProcessor也可以用ConfigurableApplicationContext以编程方式注册，只是注册的BeanFactoryPostProcessorbean将按注册顺序应用
* BeanFactoryPostProcessor不考虑@Order注解
```java
@FunctionalInterface
public interface BeanFactoryPostProcessor {

	/**
     * 1、在标准初始化后修改ApplicationContext内部的bean factory
     * 2、所有BeanDefinition定义都将被加载，但bean还没有被实例化，允许对BeanDefinition重写或添加属性，甚至对bean过早实例化
	 */
	void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException;
}
```

## BeanDefinitionRegistryPostProcessor
* 对BeanFactoryPostProcessor进行扩展，允许在BeanFactoryPostProcessor开始调用之前注册更多的BeanDefinition，甚至可以注册BeanFactoryPostProcessor实例
```java
public interface BeanDefinitionRegistryPostProcessor extends BeanFactoryPostProcessor {

	/**
     * 1、在标准初始化后修改ApplicationContext内部的BeanDefinition
     * 2、所有BeanDefinition定义都将被加载，但bean还没有被实例化，允许在BeanFactoryPostProcessor阶段前添加BeanDefinition
	 */
	void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) throws BeansException;
}
```


## BeanPostProcessor 
* 允许自定义hook修改bean的实例，在bean实例化后，初始化方法执行前、后分别执行，比如：检查baen的标记接口或用代理包装bean实例
* ApplicationContext自动在BeanDefinition中自动检测到BeanPostProcessor bean，在ApplicationContext中的BeanPostProcessor可以根据org.springframework.core.PriorityOrdered和org.springframework.core.Ordered语义进行排序
* BeanFactory允许对后处理器进行编程注册，将它们应用于BeanFactory创建的所有bean，以编程方式向BeanFactory注册的BeanPostProcessor bean将按注册顺序应用，对于以编程方式注册的后处理器，通过实现PriorityOrdered或Ordered接口表达的任何排序语义都将被忽略
* BeanPostProcessorbean不考虑@Order注解
```java
public interface BeanPostProcessor {

	/**
    * 1、在任何bean的回调（InitializingBean的afterPropertiesSet()或自定义init-method之前）初始化前
    * 2、调用时bean实例已经初始化，属性已经被填充
    * 3、返回的bean实例可能是原始bean的包装类，默认原样返回
    */
	@Nullable
	default Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
		return bean;
	}

	/**
    * 1、在任何bean的回调（InitializingBean的afterPropertiesSet()或自定义init-method之前）初始化后，比如InitializingBean的afterPropertiesSet()或自定义init-method之后
    * 2、返回的bean实例可能是原始bean的包装类，默认原样返回
    * 3、作为FactoryBean实例和FactoryBean创建对象实例的回调方法，后处理器可以通过相应的FactoryBean检查bean实例来决定是应用于FactoryBean还是应用于*    创建的对象，或者两者都应用
    * 4、InstantiationAwareBeanPostProcessor.postProcessBeforeInstantiation()后也将调用该方法
    */
	@Nullable
	default Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
		return bean;
	}
}
```
* 在Bean中可以通过@PostConstruct注解来指定在被Construct之后紧接着做一些初始化操作, postProcessAfterInitialization()是在@PostConstruct之后被调用的

## InstantiationAwareBeanPostProcessor
* 可以在实例化Bean前（调用postProcessBeforeInstantiation方法）、后(postProcessAfterInstantiation)提供扩展的回调接口

## SmartInstantiationAwareBeanPostProcessor
* 拓展了InstantiationAwareBeanPostProcessor接口，主要是供spring内部使用

## MergedBeanDefinitionPostProcessor
* 实例化后执行，主要将那些元数据缓存起来以提供后续的postProcessPropertyValues输入注入时获取

## DestructionAwareBeanPostProcessor
* 对象销毁的前置回调

## InitializingBean/init-method
* BeanFactory设置完所有属性后调用InitializingBean实例对象 
* 实现InitializingBean的另一种方法是指定自定义init方法
```java
public interface InitializingBean {
	/**
     * 1、BeanFactory在设置了所有bean属性并满足BeanFactoryAware、ApplicationContextAware等要求后调用
     * 2、与BeanPostProcessor结合来看, afterPropertiesSet方法将在postProcessBeforeInitialization和postProcessAfterInitialization之间被调用
	 */
	void afterPropertiesSet() throws Exception;
}
```

# spring连载：扩展点调用流程

* @Autowired 注解的处理类 AutowiredAnnotationBeanPostProcessor
* ConfigurationClassPostProcessor拓展BeanDefinitionRegistryPostProcessor 解析@Configuration配置类中的@Import、@PropertySource、@ComponentScan、@ImportResource、@Bean等
* @Autowired 的设计，就是用来完成对象的属性注入的。对与对象之间依赖关系的处理是在 org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#populateBean(String beanName, RootBeanDefinition mbd, @Nullable BeanWrapper bw) 这个方法中来处理的
* Aware接口调用AbstractAutowireCapableBeanFactory#invokeAwareMethods，其他的Aware接口呢？通过 ApplicationContextAwareProcessor
* ServletContextAware 是什么时候注入的呢？ AbstractRefreshableWebApplicationContext 继承了  AbstractApplicationContext类，重写了 postProcessBeanFactory方法；这就是使用了Spring的钩子方法。

* BeanPostProcessor调用点：
    调用类AbstractAutowireCapableBeanFactory.initializeBean(String beanName, Object bean, @Nullable RootBeanDefinition mbd)
        * invokeAwareMethods() 调用实现Aware接口方法
        * applyBeanPostProcessorsBeforeInitialization()  实现BeanPostProcessor类 的 postProcessBeforeInitialization方法
        * invokeInitMethods() 如果bean实现了InitializingBean 接口，则调用afterPropertiesSet方法，如果有配置init-method,则调用配置的初始化方法
        * applyBeanPostProcessorsAfterInitialization() 实现BeanPostProcessor类 的 postProcessAfterInitialization方法

BeanFactoryPostProcessor调用点：AbstractApplicationContext#refresh 


Spring 中定义了 3 种自定义初始化 和 销毁方法
通过@Bean指定init-method和destroy-method属性
Bean实现InitializingBean（定义初始化逻辑），DisposableBean（定义销毁逻辑）;
@PostConstruct：在bean创建完成并且属性赋值完成；来执行初始化方法，@PreDestroy：在容器销毁bean之前通知我们进行清理工作。   
InitDestroyAnnotationBeanPostProcessor的作用就是让这种方式生效


InitializingBean调用点：
* 为实现该接口的bean提供默认的初始化方法，也可以在xml配置bean的使用init-method来实现初始化方法
第一：实现InitializingBean接口，继而实现afterPropertiesSet的方法
第二：反射原理，配置文件使用init-method标签直接注入bean
* 调用点
    调用类AbstractAutowireCapableBeanFactory.invokeInitMethods(String beanName, Object bean, @Nullable RootBeanDefinition mbd)
* 应用场景
    * 在SpringMVC中AbstractHandlerMethodMapping就实现了InitializingBean接口，当一个RequestMappingHandlerMapping的实例创建完成后会接着调用afterPropertiesSet方法，扫描所有的controller完成所有的handler method的注册。
    * TransactionTemplate，判断transactionManager是否初始化，没有初始化抛异常

* 执行过程：构造方法-->BeanPostProcessor-->InitializingBean-->bean中的初始化方法
