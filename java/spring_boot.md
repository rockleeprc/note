### Profile

* 通过`@Bean`和`@Profile("dev")`配置环境所需要的行为
* 使用Environment中的`ActiveProfiles`设置参数

### Bean的初始化/销毁

* Java 配置：在方法上配置`@Bean(initMethod = "init", destroyMethod = "destroy")`指定初始化方法，销毁方法
* JSR250：在方法上配置`@PostConstruct`和`@PreDestroy`

**initMethod、@PostConstruct**在构造方法执行后调用

**destroyMethod、@PreDestroy **在bean销毁前执行

### 外部配置

* 命令行参数：`java -jar xx.jar --server.port=8888` 
* 注解：

  * `@PropertySource`指定properties路径
  * `@Value`注入到field中
* 类型安全配置：

  * `@ConfigurationProperties`指定properties前缀
  * `@PropertySource`指定properties路径
  * `@Configuration`声明这是一个配置类

### @Enable*原理

使用@Import导入配置类

* 直接导入，如@EnableScheduling中`@Import(SchedulingConfiguration.class)`，配置类SchedulingConfiguration中只使用@Configuration+@Bean
* 依据条件选择配置类，如EnableAsync中`@Import(AsyncConfigurationSelector.class)`，配置类继承`AdviceModeImportSelector`实现`selectImports()`
* 动态注入bean，如EnableAspectJAutoProxy中`@Import(AspectJAutoProxyRegistrar.class)`，配置类实现ImportBeanDefinitionRegistrar的作用是在运行时自动添加Bean到已有的配置类中

### SpringMVC配置

* 配置类继承`WebMvcConfigurerAdapter`

* 注解`@EnableWebMvc`开启对springMVC配置支持

* 使用`@Bean`让spring管理对象

* 静态资源映射

  * org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter.addResourceHandlers(ResourceHandlerRegistry)

* 拦截器

  * org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter.addInterceptors(InterceptorRegistry)

* 页面跳转（无业务逻辑）

  * org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter.addViewControllers(ViewControllerRegistry)

* 转换器

  * org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter.extendMessageConverters(List<HttpMessageConverter<?>>)
  * org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter.configureMessageConverters(List<HttpMessageConverter<?>>)

* 参数路径（不忽略.以后的内容）

  * org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter.configurePathMatch(PathMatchConfigurer)

  
### Spring boot入口类

* @SpringBootApplication标记该类是一个Spring boot入口类

  ```java
  @Target(ElementType.TYPE)
  @Retention(RetentionPolicy.RUNTIME)
  @Documented
  @Inherited
  @SpringBootConfiguration
  @EnableAutoConfiguration//根据jar包依赖自动进行配置
  @ComponentScan(excludeFilters = {
  		@Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
  		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
  public @interface SpringBootApplication {}
  ```

* Spring boot扫描@SpringBootApplication所在类的同级包，及子包下的Bean

### Profile配置

* 定义多个配置文件application-dev.properties、application-prod.properties
* 在application.properties中指定spring.profiles.active=dev使用哪个配置文件



### Spring boot web

#### web相关配置

* 于web相关的自动配置在spring-boot-autoconfigure-1.5.13.RELEASE.jar的org.springframework.boot.autoconfigure.web包下
* Springmvc 自动配置类org.springframework.boot.autoconfigure.web.WebMvcAutoConfiguration
  * viewResolver(BeanFactory beanFactory)
  * addResourceHandlers(ResourceHandlerRegistry registry) 静态资源
    * /static
    * /public
    * /resources
    * /METE-INF/resourdces
  *  addFormatters(FormatterRegistry registry)
  * configureMessageConverters(List<HttpMessageConverter<?>> converters)

##### 扩展配置

* 三种方式
  * org.springframework.web.servlet.config.annotation.WebMvcConfigurer
  * org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter，无需使用@EnableWebMvc注解
  * org.springframework.web.servlet.config.annotation.WebMvcConfigurationSupport
* 注册Servlet、Filter、Listener
  * 通过@Configuration+@Bean
  * ServletRegistrantionBean、FilterRegistrantionBean、ServletListenerRegistrantionBean

##### 容器配置

*  容器参数置类org.springframework.boot.autoconfigure.web.ServerProperties
* jetty 替换tomcat



### Spring boot data

##### 事务支持

* 自动配置事务管理器
  * org.springframework.boot.autoconfigure.jdbc.DataSourceTransactionManagerAutoConfiguration

##### 缓存

* spring定义CacheManager和Cache接口来统一不同的缓存技术