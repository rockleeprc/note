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



### 入口类

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



