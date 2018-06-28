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

  



