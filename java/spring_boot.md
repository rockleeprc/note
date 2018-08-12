# 一、

# 二、配置文件

## 1、yaml语法

### 1）字面量：

* 字符串默认不用加上单引号或者双引号
* 双引号：不会转义字符串里面的特殊字符；特殊字符会作为本身想表示的意思
  * name:   "zhangsan \n lisi"，输出 zhangsan 换行  lisi
* 单引号；会转义特殊字符，特殊字符最终只是一个普通的字符串数据
  * name:   ‘zhangsan \n lisi’，输出 zhangsan \n  lisi

### 2）复杂对象映射:

java bean

```java
public class Car {
    private String brand;
    private Integer number;
}
public class Person {
    private String name;
    private Integer age;
    private Boolean boss;
    private Date birth;
}
```

yaml配置

```yaml
person:
    name: zhangsan
    age: 18
    boss: false
    birth: 2017/12/12
    maps: {k1: v1,k2: 12}
    cars:
      - brand: beanz
        number: 1234
      - brand: bmw
        number: 4321
    pets:
      - 小狗
      - 猫
```

### 3）对象绑定

* @ConfigurationProperties：类中的属性和配置文件中的配置进行绑定，默认从全局配置文件中获取配置
* @Component：对象实例交给容器管理，只有对象在容器中才能使用配置绑定功能功能

```java
@Component
@ConfigurationProperties(prefix = "person")
public class Person {}
```



使yaml配置文件有提示功能，倒入以下配置，但是在ided中部起作用，spring官方文旦B.3

```xml
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-configuration-processor</artifactId>
			<optional>true</optional>
		</dependency>
```



## 5、profile

### 1）拆分多个文件

* application.yml/properties
* application-dev.yml/properties
* application-prod.yml/properties

# 2）yml文档块	

使用`---`区分文档块，使用`spring.profiles=dev`声明该文档块属于哪个profile

```yaml
server:
  port: 8080
spring:
  profiles:
    active:
      - dev

---
server:
  port: 8081
spring:
  profiles:
      - dev

---
server:
  port: 8082
spring:
  profiles:
      - prod

```

### 3）激活profile

* 在application.yml/properties主配置文件中指定

```yam
spring:
  profiles:
    active: dev
```

* 命令行参数指定

```shell
java -jar spring-boot-profile-0.0.1-SNAPSHOT.jar --spring.profiles.active=prod
```

* 虚拟机参数

```she
-Dspring.profiles.active=dev
```

## 6、配置文件加载位置

springboot启动时会扫描以下位置的application.properties/application.yml作为配置文件

* –file:./config/
* –file:./
* –classpath:/config/
* –classpath:/

优先级由高到底，相同配置，高优先级的配置会覆盖低优先级的配置，不同配置高低互补

也可以通过命令行指定外部配置文件，配置内容相同的覆盖，不同的互补

```shell
java -jar spring-boot-profile-0.0.1-SNAPSHOT.jar --spring.config.location=D:/application.yml
```



## 8、自动配置原理

### 1）@SpringBootApplication作用

标注这是一个主程序类，说明这个一个spring boot应用

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration	//标注在类上，表示这是一个spring boot配置类，@Configuration
@EnableAutoConfiguration	//开启自动配置功能
@ComponentScan(excludeFilters = {
		@Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {}
```

### 2）自动配置类加载过程

@EnableAutoConfiguration开启启动配置功能，引入EnableAutoConfigurationImportSelector，这也是启动配置的入口类

```java
@SuppressWarnings("deprecation")
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(EnableAutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {}
```

加载配置相关的最接口，在AutoConfigurationImportSelector中实现

```java
public interface ImportSelector {
	String[] selectImports(AnnotationMetadata importingClassMetadata);
}
```

扫描所有classpath下jar中的**META-INF/spring.factories**文件，通过springSPI找到key为org.springframework.boot.autoconfigure.EnableAutoConfiguration的所有值，并添加到容器中

```java
List<String> configurations = SpringFactoriesLoader.loadFactoryNames(	getSpringFactoriesLoaderFactoryClass(), getBeanClassLoader());
```

org.springframework.boot.autoconfigure.EnableAutoConfiguration对应的内容

```properties
# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,\
org.springframework.boot.autoconfigure.batch.BatchAutoConfiguration,\
org.springframework.boot.autoconfigure.cache.CacheAutoConfiguration,\
org.springframework.boot.autoconfigure.cassandra.CassandraAutoConfiguration,\
org.springframework.boot.autoconfigure.cloud.CloudAutoConfiguration,\
org.springframework.boot.autoconfigure.context.ConfigurationPropertiesAutoConfiguration,\
org.springframework.boot.autoconfigure.context.MessageSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.context.PropertyPlaceholderAutoConfiguration,\
org.springframework.boot.autoconfigure.couchbase.CouchbaseAutoConfiguration,\
org.springframework.boot.autoconfigure.dao.PersistenceExceptionTranslationAutoConfiguration,\
org.springframework.boot.autoconfigure.data.cassandra.CassandraDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.cassandra.CassandraRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.couchbase.CouchbaseDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.couchbase.CouchbaseRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchAutoConfiguration,\
org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.jpa.JpaRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.ldap.LdapDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.ldap.LdapRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.mongo.MongoDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.mongo.MongoRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.neo4j.Neo4jDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.neo4j.Neo4jRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.solr.SolrRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.redis.RedisAutoConfiguration,\
org.springframework.boot.autoconfigure.data.redis.RedisRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.rest.RepositoryRestMvcAutoConfiguration,\
org.springframework.boot.autoconfigure.data.web.SpringDataWebAutoConfiguration,\
org.springframework.boot.autoconfigure.elasticsearch.jest.JestAutoConfiguration,\
org.springframework.boot.autoconfigure.freemarker.FreeMarkerAutoConfiguration,\
org.springframework.boot.autoconfigure.gson.GsonAutoConfiguration,\
org.springframework.boot.autoconfigure.h2.H2ConsoleAutoConfiguration,\
org.springframework.boot.autoconfigure.hateoas.HypermediaAutoConfiguration,\
org.springframework.boot.autoconfigure.hazelcast.HazelcastAutoConfiguration,\
org.springframework.boot.autoconfigure.hazelcast.HazelcastJpaDependencyAutoConfiguration,\
org.springframework.boot.autoconfigure.info.ProjectInfoAutoConfiguration,\
org.springframework.boot.autoconfigure.integration.IntegrationAutoConfiguration,\
org.springframework.boot.autoconfigure.jackson.JacksonAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.JdbcTemplateAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.JndiDataSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.XADataSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.DataSourceTransactionManagerAutoConfiguration,\
org.springframework.boot.autoconfigure.jms.JmsAutoConfiguration,\
org.springframework.boot.autoconfigure.jmx.JmxAutoConfiguration,\
org.springframework.boot.autoconfigure.jms.JndiConnectionFactoryAutoConfiguration,\
org.springframework.boot.autoconfigure.jms.activemq.ActiveMQAutoConfiguration,\
org.springframework.boot.autoconfigure.jms.artemis.ArtemisAutoConfiguration,\
org.springframework.boot.autoconfigure.flyway.FlywayAutoConfiguration,\
org.springframework.boot.autoconfigure.groovy.template.GroovyTemplateAutoConfiguration,\
org.springframework.boot.autoconfigure.jersey.JerseyAutoConfiguration,\
org.springframework.boot.autoconfigure.jooq.JooqAutoConfiguration,\
org.springframework.boot.autoconfigure.kafka.KafkaAutoConfiguration,\
org.springframework.boot.autoconfigure.ldap.embedded.EmbeddedLdapAutoConfiguration,\
org.springframework.boot.autoconfigure.ldap.LdapAutoConfiguration,\
org.springframework.boot.autoconfigure.liquibase.LiquibaseAutoConfiguration,\
org.springframework.boot.autoconfigure.mail.MailSenderAutoConfiguration,\
org.springframework.boot.autoconfigure.mail.MailSenderValidatorAutoConfiguration,\
org.springframework.boot.autoconfigure.mobile.DeviceResolverAutoConfiguration,\
org.springframework.boot.autoconfigure.mobile.DeviceDelegatingViewResolverAutoConfiguration,\
org.springframework.boot.autoconfigure.mobile.SitePreferenceAutoConfiguration,\
org.springframework.boot.autoconfigure.mongo.embedded.EmbeddedMongoAutoConfiguration,\
org.springframework.boot.autoconfigure.mongo.MongoAutoConfiguration,\
org.springframework.boot.autoconfigure.mustache.MustacheAutoConfiguration,\
org.springframework.boot.autoconfigure.orm.jpa.HibernateJpaAutoConfiguration,\
org.springframework.boot.autoconfigure.reactor.ReactorAutoConfiguration,\
org.springframework.boot.autoconfigure.security.SecurityAutoConfiguration,\
org.springframework.boot.autoconfigure.security.SecurityFilterAutoConfiguration,\
org.springframework.boot.autoconfigure.security.FallbackWebSecurityAutoConfiguration,\
org.springframework.boot.autoconfigure.security.oauth2.OAuth2AutoConfiguration,\
org.springframework.boot.autoconfigure.sendgrid.SendGridAutoConfiguration,\
org.springframework.boot.autoconfigure.session.SessionAutoConfiguration,\
org.springframework.boot.autoconfigure.social.SocialWebAutoConfiguration,\
org.springframework.boot.autoconfigure.social.FacebookAutoConfiguration,\
org.springframework.boot.autoconfigure.social.LinkedInAutoConfiguration,\
org.springframework.boot.autoconfigure.social.TwitterAutoConfiguration,\
org.springframework.boot.autoconfigure.solr.SolrAutoConfiguration,\
org.springframework.boot.autoconfigure.thymeleaf.ThymeleafAutoConfiguration,\
org.springframework.boot.autoconfigure.transaction.TransactionAutoConfiguration,\
org.springframework.boot.autoconfigure.transaction.jta.JtaAutoConfiguration,\
org.springframework.boot.autoconfigure.validation.ValidationAutoConfiguration,\
org.springframework.boot.autoconfigure.web.DispatcherServletAutoConfiguration,\
org.springframework.boot.autoconfigure.web.EmbeddedServletContainerAutoConfiguration,\
org.springframework.boot.autoconfigure.web.ErrorMvcAutoConfiguration,\
org.springframework.boot.autoconfigure.web.HttpEncodingAutoConfiguration,\
org.springframework.boot.autoconfigure.web.HttpMessageConvertersAutoConfiguration,\
org.springframework.boot.autoconfigure.web.MultipartAutoConfiguration,\
org.springframework.boot.autoconfigure.web.ServerPropertiesAutoConfiguration,\
org.springframework.boot.autoconfigure.web.WebClientAutoConfiguration,\
org.springframework.boot.autoconfigure.web.WebMvcAutoConfiguration,\
org.springframework.boot.autoconfigure.websocket.WebSocketAutoConfiguration,\
org.springframework.boot.autoconfigure.websocket.WebSocketMessagingAutoConfiguration,\
org.springframework.boot.autoconfigure.webservices.WebServicesAutoConfiguration
```

### 3）XXXAutoConfiguration

```java
//这是一个配置类
@Configuration
//开启properties配置，将配置文件的值和HttpEncodingProperties中的属性绑定，并把HttpEncodingProperties加载到IoC容器中
@EnableConfigurationProperties(HttpEncodingProperties.class)	
//当前是web应用生效
@ConditionalOnWebApplication	
//classpath中有CharacterEncodingFilter生效
@ConditionalOnClass(CharacterEncodingFilter.class)	
//是否配置spring.http.encoding.enabled，如果没有也默认生效spring.http.encoding.enabled=true
@ConditionalOnProperty(prefix = "spring.http.encoding", value = "enabled", matchIfMissing = true)	
public class HttpEncodingAutoConfiguration {
    
	private final HttpEncodingProperties properties;
	//注入HttpEncodingProperties
	public HttpEncodingAutoConfiguration(HttpEncodingProperties properties) {
		this.properties = properties;
	}
	
    //方法返回值添加到容器中
	@Bean
    //容器中没CharacterEncodingFilter个实例时添加
	@ConditionalOnMissingBean(CharacterEncodingFilter.class)
	public CharacterEncodingFilter characterEncodingFilter() {
		CharacterEncodingFilter filter = new OrderedCharacterEncodingFilter();
		filter.setEncoding(this.properties.getCharset().name());
		filter.setForceRequestEncoding(this.properties.shouldForce(Type.REQUEST));
		filter.setForceResponseEncoding(this.properties.shouldForce(Type.RESPONSE));
		return filter;
	}
}
```

根据Conditional条件，判断这个类是否自动配置，如果自动配置，将配置的类实例化后放入IoC容器

配置类的参数从配置文件（properties、yaml）中获取



在配置文件中配置 **debug=true**，让控制台打印自动配置报告

```properties
=========================
AUTO-CONFIGURATION REPORT
=========================


Positive matches:（自动配置类启用的）
-----------------

   DispatcherServletAutoConfiguration matched:
      - @ConditionalOnClass found required class 'org.springframework.web.servlet.DispatcherServlet'; @ConditionalOnMissingClass did not find unwanted class (OnClassCondition)
      - @ConditionalOnWebApplication (required) found StandardServletEnvironment (OnWebApplicationCondition)
        
    
Negative matches:（没有启动，没有匹配成功的自动配置类）
-----------------

   ActiveMQAutoConfiguration:
      Did not match:
         - @ConditionalOnClass did not find required classes 'javax.jms.ConnectionFactory', 'org.apache.activemq.ActiveMQConnectionFactory' (OnClassCondition)

   AopAutoConfiguration:
      Did not match:
         - @ConditionalOnClass did not find required classes 'org.aspectj.lang.annotation.Aspect', 'org.aspectj.lang.reflect.Advice' (OnClassCondition)
        
```



### 4）@Conditional派生注解

作用：必须是@Conditional指定的条件成立，才给容器中添加组件，配置配里面的所有内容才生效；

| @Conditional扩展注解            | 作用（判断是否满足当前指定条件）                 |
| ------------------------------- | ------------------------------------------------ |
| @ConditionalOnJava              | 系统的java版本是否符合要求                       |
| @ConditionalOnBean              | 容器中存在指定Bean；                             |
| @ConditionalOnMissingBean       | 容器中不存在指定Bean；                           |
| @ConditionalOnExpression        | 满足SpEL表达式指定                               |
| @ConditionalOnClass             | 系统中有指定的类                                 |
| @ConditionalOnMissingClass      | 系统中没有指定的类                               |
| @ConditionalOnSingleCandidate   | 容器中只有一个指定的Bean，或者这个Bean是首选Bean |
| @ConditionalOnProperty          | 系统中指定的属性是否有指定的值                   |
| @ConditionalOnResource          | 类路径下是否存在指定资源文件                     |
| @ConditionalOnWebApplication    | 当前是web环境                                    |
| @ConditionalOnNotWebApplication | 当前不是web环境                                  |
| @ConditionalOnJndi              | JNDI存在指定项                                   |



# 三、日志

## 1、日志框架

### 1）日志门面

* JCL（Jakarta  Commons Logging）    
* SLF4j（Simple  Logging Facade for Java）
* jboss-logging

### 2）日志实现

* Log4j 
* Log4j2
* JUL（java.util.logging）
*   Logback



## 2、slf4j使用

* 引入slf4j
* 删除框架自己的日志依赖
* 引入桥接slf4j包
  * jcl-over-slf4j-1.7.25.jar
  * log4j-over-slf4j-1.7.25.jar
  * log4j-over-slf4j-1.7.25.jar
* 引入slf4j实现包

## 3、springboot中的日志实现

### 1）pom依赖

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-core</artifactId>
    <version>${spring.version}</version>
    <exclusions>
        <exclusion>
            <groupId>commons-logging</groupId>
            <artifactId>commons-logging</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

```
<artifactId>spring-boot-starter-logging</artifactId>
<dependencies>
    <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-classic</artifactId>
    </dependency>
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>jcl-over-slf4j</artifactId>
    </dependency>
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>jul-to-slf4j</artifactId>
    </dependency>
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>log4j-over-slf4j</artifactId>
    </dependency>
</dependencies>
```

### 2）日志级别

* trace
* debug
* info（springboot 默认级别）
* warn
* error

### 3）日志配置

* logging.path：指定输出文件的路径，默认文件名称为spring.log
* logging.file：指定输出文件的路径和文件名



# 四、Web开发

## 4、SpringMVC自动配置





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