## 组件注入

### @Bean+@Configuration

* 在类上标注@Configuration
* 在方法上标注@Bean

### @ComponentScan

* 扫描指定路径下的类

  ```java
  @ComponentScan(basePackages = "com.foo")
  ```

* 不包含某个注解

  ```java
  @ComponentScan(basePackages="com.foo",excludeFilters = {@ComponentScan.Filter({Controller.class,Service.class})})
  ```

* 只包含某个注解，useDefaultFilters一定要配置为false，不使用默认的过滤规则

  ```java
  @ComponentScan(basePackages = "com.foo",includeFilters = {@ComponentScan.Filter({Repository.class})},useDefaultFilters=false)
  完整写法
  @ComponentScan(basePackages = "com.foo",includeFilters =   {@ComponentScan.Filter(type=FilterType.ANNOTATION,classes={Repository.class})},useDefaultFilters=false)
  ```

### @Conditional

* 有条件的添加bean，实现Condition接口

* matches()方法可以获得beanFactory、classLoader、beanDefinitionRegistry、environment等基础信息

  ```java
  public class WindowsCondition implements Condition {
      public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
          ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
          ClassLoader classLoader = context.getClassLoader();
          BeanDefinitionRegistry registry = context.getRegistry();
          ResourceLoader resourceLoader = context.getResourceLoader();
          Environment environment = context.getEnvironment();
  
          String os = environment.getProperty("os.name");
          if (os.contains("Windows")) {
              return true;
          }
  
          return false;
      }
  }
  ```

### @Import

* 导入一个类

  ```java
  @Import({Red.class})
  ```

* 导入一个实现ImportSelector接口的类，可以导入多个类，类必须为全路径（Springboot中大量应用）

  ```java
  public class MyImport implements ImportSelector {
      public String[] selectImports(AnnotationMetadata importingClassMetadata) {
          return new String[]{"com.foo.bean.Blue","com.foo.bean.Yellow"};
      }
  }
  ```

* 导入一个实现ImportBeanDefinitionRegistrar接口的类，使用beanDefinitionRegistry手动注册一个bean

  ```java
  public class MyImportRegister implements ImportBeanDefinitionRegistrar {
      public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
          registry.registerBeanDefinition("black", new RootBeanDefinition(Black.class));
      }
  }
  ```

### FactoryBean

* 实现FactoryBean接口，通过@Bean注入FactoryBean（不是bean本身，但获取的是bean实例），用`&Bean`要获取FactoryBean

  ```java
  public class GreenFactoryBean implements FactoryBean<Green> {
      public Green getObject() throws Exception {
          System.out.println("...........getObject()");
          return new Green();
      }
  
      public Class<?> getObjectType() {
          System.out.println("...........getObjectType()");
          return Green.class;
      }
  
      public boolean isSingleton() {
          return true;
      }
  }
  //配置类
  @Bean
  public GreenFactoryBean green() {
      return new GreenFactoryBean();
  }
  ```

## Bean生命周期

### @Bean

* 在@Bean注解中指定bean的初始化、销毁方法

  ```java
  @Bean(initMethod = "init", destroyMethod = "destroy")
  ```

* 单例对象：BeanFactory创建对象-->initMethod-->destroyMethod（容器关闭）
* prototype对象：BeanFactory创建对象--->initMethod，没有destroyMethod过程，对象的销毁不由容器控制
* 单例对象容器启动后就会创建，prototype对象只有在获取对象时才创建

### InitializingBean、DisposableBean

* 实现InitializingBean.afterPropertiesSet()方法，完成bean的初始化
* 实现DisposableBean.destroy()方法，完成bean的销毁
* bean的创建过程与@Bean方式一样

### JSR250：@PostConstruct、@PreDestroy

* javax包的注解，不是spring的
* bean的创建过程与@Bean方式一样

### BeanPostProcessor

* 实现BeanPostProcessor接口，在Bean实例化后，init方法前调用`postProcessBeforeInitialization()`，destory方法前`postProcessAfterInitialization()`
* bean生命周期流程
  * Bean Constructor
  * postProcessBeforeInitialization
  * afterPropertiesSet/@Bean initMethod/@PostConstruct
  * postProcessAfterInitialization
  * destroyMethod/@Bean destroyMethod/@PreDestroy
* 源码调用过程
  * org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory
    * doCreateBean(); 	//创建bean
      * populateBean(beanName, mbd, instanceWrapper);	//给bean的属性赋值
      * initializeBean(beanName, exposedObject, mbd);
        * wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName)	;//调用beanPostProcessor
        * invokeInitMethods(beanName, wrappedBean, mbd); //执行自定义初始化
        * wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);//调用beanPostProcessor
* 应用场景
  * ApplicationContextAwareProcessor：xxxAware接口注入
  * BeanValidationPostProcessor：bean数据校验
  * InitDestroyAnnotationBeanPostProcessor：处理JSR250注解
  * AutowiredAnnotationBeanPostProcessor：自动装配
  * AsyncAnnotationBeanPostProcessor：异步方法处理

## 属性赋值

### @Value、@PropertySource

* 基本数值

  ```java
  @Value("lisi")
  private String name;
  ```

* SpEL

  ```java
  @Value("#{20+20}")
  private Integer age;
  ```

* ${}

  ```java
  @Value("${person.address}")
  private String address;
  ```

* @PropertySource加载*.propertie文件

  ```java
  @PropertySource(value = "classpath:person.properties")
  ```

* propertie中的内容也可以使用ApplicationConetext-->ConfigurableEnvironment获取配置文件内容

## 自动装配

### @Autowired

* required，如果容器中没有需要装备的bean会抛异常，设置required=false，没有组件就不装配

  ```java
  @Autowired(required = false)
  ```

* 标注位置
  * 属性
  * 方法
  * 方法参数
  * 构造器，如果组件只有一个有参数构造器，@Autowired可以省略
  * @Bean+方法参数，@Autowired可以省略

### @Qualifier

* 指定装配bean的名称

  ```java
  @Qualifier("cat1")
  @Autowired(required = false)
  private Cat cat;
  ```

### @Primary

* 当装备的对象在容器中有多个时，优先使用@Primary标记的，机器猫将装配

  ```java
  @Bean
  public Cat cat1() {
      return new Cat("加菲猫");
  }
  
  @Bean
  @Primary
  public Cat cat2() {
      return new Cat("机器猫");
  }
  ```

### 匹配顺序

- @Qualifier，优先级最高
- @Primary，多个实例时，优先使用
- 类型，@Autowired先匹配类型
- 属性名称，@Autowired在匹配属性名称

### @Resource(JSR250)

* 和@Autowired一样实现自动装配，默认按照属性名称
* 不支持@Primary
* 不支持@Autowired(required = false)

### @Inject(JSR330)

* 和@Autowired一样实现自动装配
* 需要单独导入javax.inject包
* 支持@Primary
* 不支持@Autowired(required = false)

### @Profile

* 根据配置使用不同的bean装配

  ```java
  @Profile("test")
  @Bean
  public Cat catTest() {
      return new Cat("Test");
  }
  
  @Profile("dev")
  @Bean
  public Cat catDev() {
      return new Cat("Dev");
  }
  
  @Profile("prod")
  @Bean
  public Cat catProd() {
      return new Cat("Prod");
  }
  ```

* @Profile生效方法

  * 配置启动参数

    ```java
    -Dspring.profiles.active=test
    ```

  * 编码

    ```java
    AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();
    context.getEnvironment().setActiveProfiles("prod");
    context.register(ProfileConfiguration.class);
    context.refresh();
    ```











