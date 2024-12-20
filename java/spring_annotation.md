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

* 自定义扫描类

  ```java
  @ComponentScan(basePackages = "com.foo", includeFilters = {@ComponentScan.Filter(type = FilterType.CUSTOM, classes={MyTypeFilter.class})}, useDefaultFilters = false)
  
  public class MyTypeFilter implements TypeFilter {
      @Override
      public boolean match(MetadataReader metadataReader, MetadataReaderFactory metadataReaderFactory) throws IOException {
          ClassMetadata classMetadata = metadataReader.getClassMetadata();
          if(classMetadata.getClassName().contains("Service")|| classMetadata.getClassName().contains("Dao")){
              return true;
          }
          return false;
      }
  }
  ```

* 配置bean的sceop

  ```java
  @Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
  ```

  

### @Conditional

* 有条件的添加bean，实现Condition接口

* matches()方法可以获得beanFactory、classLoader、beanDefinitionRegistry、environment等基础信息

* 组件注入时会根据matches()返回值判断是否注入

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

* springboot Conditional

  | 条件化注解                      | 配置生效条件                                         |
  | ------------------------------- | ---------------------------------------------------- |
  | @ConditionalOnBean              | 配置了某个特定bean                                   |
  | @ConditionalOnMissingBean       | 没有配置特定的bean                                   |
  | @ConditionalOnClass             | Classpath里有指定的类                                |
  | @ConditionalOnMissingClass      | Classpath里没有指定的类                              |
  | @ConditionalOnExpression        | 给定的Spring Expression Language表达式计算结果为true |
  | @ConditionalOnJava              | Java的版本匹配特定指或者一个范围值                   |
  | @ConditionalOnProperty          | 指定的配置属性要有一个明确的值                       |
  | @ConditionalOnResource          | Classpath里有指定的资源                              |
  | @ConditionalOnWebApplication    | 这是一个Web应用程序                                  |
  | @ConditionalOnNotWebApplication | 这不是一个Web应用程序                                |

### @Import

* 导入一个类

  ```java
  @Import({Red.class})
  ```

* 导入实现ImportSelector接口的类，可以导入多个类，类必须为全路径（Springboot中大量应用）

  ```java
  public class MyImport implements ImportSelector {
      public String[] selectImports(AnnotationMetadata importingClassMetadata) {
          return new String[]{"com.foo.bean.Blue","com.foo.bean.Yellow"};
      }
  }
  ```

* 导入实现ImportBeanDefinitionRegistrar接口的类，使用beanDefinitionRegistry手动注册一个bean

  ```java
  public class MyImportRegister implements ImportBeanDefinitionRegistrar {
      public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
          registry.registerBeanDefinition("black", new RootBeanDefinition(Black.class));
      }
  }
  ```

### FactoryBean

* 实现FactoryBean接口，通过@Bean注入FactoryBean（不是bean本身，但获取的是bean实例）

* 获取bean实例时用@Bean标注的方法名称`green`，使用`&green`要获取FactoryBean实例

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

* 实现BeanPostProcessor接口，在Bean实例化后，自定义init方法前调用`postProcessBeforeInitialization()`，自定义destory方法前`postProcessAfterInitialization()`
* bean生命周期流程
  * Bean Constructor
  * postProcessBeforeInitialization
  * afterPropertiesSet/@Bean initMethod/@PostConstruct（自定义初始化方法）
  * postProcessAfterInitialization
  * destroyMethod/@Bean destroyMethod/@PreDestroy（自定义销毁方法）
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
- 属性名称，@Autowired在类型不匹配时，匹配属性名称

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

## 通知

### @Aspect

* 标注一个类上，声明一个切面类

### @Pointcut

* 标注在方法上，声明一个连接点，方法没有任何实现

  ```java
  @Pointcut("execution(public Integer com.foo.service.CalcService.*(..))")
  ```

### @Before、@After、@AfterReturning、@Around、@AfterThrowing

* 引用本类中的切点，直接使用方法名

  ```java
   @Before("pointCut()")
  ```

* 引用其他类的切点使用完整的类名

  ```java
  @After("com.foo.aspect.LogAspect.pointCut()")
  ```

* 标注通知的方法中要引用JoinPoint时，该参数必须位于参数列表的首位

### @EnableAspectJAutoProxy

* 配置类要标注开启AspectJ

* 业务组件和切面都需要注册到spring容器中

* 这个自动配置类导入一个实现ImportBeanDefinitionRegistrar接口的实现类，向容器中注入了AnnotationAwareAspectJAutoProxyCreator.class

  ```java
  @Import(AspectJAutoProxyRegistrar.class)
  ```

* AnnotationAwareAspectJAutoProxyCreator最终通过BeanPostProcessor、BeanFactoryAware实现AOP功能

  ```java
  public abstract class AbstractAutoProxyCreator extends ProxyProcessorSupport
  		implements SmartInstantiationAwareBeanPostProcessor, BeanFactoryAware {}
  ```

### 通知完整例子

```java
@Aspect
public class LogAspect {

    @Pointcut("execution(public Integer com.foo.service.CalcService.*(..))")
    public void pointCut() {
    }

    @Before("pointCut()")
    public void before(JoinPoint joinPoint) {
        String methodName = joinPoint.getSignature().getName();
        System.out.println("LogAspect.before..." + methodName);
    }

    @AfterReturning(value = "pointCut()", returning = "result")
    public void afterReturning(JoinPoint joinPoint, Integer result) {
        String methodName = joinPoint.getSignature().getName();
        System.out.println("LogAspect.before..." + methodName + " 返回值:" + result);
    }

    @After("com.foo.aspect.LogAspect.pointCut()")
    public void after(JoinPoint joinPoint) {
        String methodName = joinPoint.getSignature().getName();
        System.out.println("LogAspect.after..." + methodName);
    }

    @Around("pointCut()")
    public void around(ProceedingJoinPoint pjp) throws Throwable {
        System.out.println("before around ");
        System.out.println("around "+pjp.getTarget().getClass());
        Object[] args = pjp.getArgs();
        System.out.println("around "+args[0]);
        pjp.proceed();
        System.out.println("after around ");
    }


    @AfterThrowing(value = "pointCut()", throwing = "exception")
    public void afterThrowing(JoinPoint joinPoint, Exception exception) {
        String methodName = joinPoint.getSignature().getName();
        System.out.println("LogAspect.before..." + methodName+" exception:"+exception.getMessage());
    }
}
@Configuration
@EnableAspectJAutoProxy
public class AopConfiguration {

    @Bean
    public CalcService calcService() {
        return new CalcService();
    }

    @Bean
    public LogAspect logAspect() {
        return new LogAspect();
    }
}
```

## 源码分析

### @EnableAspectJAutoProxy装配流程

```java
AnnotationAwareAspectJAutoProxyCreator#initBeanFactory
	AbstractAdvisorAutoProxyCreator#setBeanFactory
    AbstractAdvisorAutoProxyCreator#initBeanFactory
    	AbstractAutoProxyCreator#setBeanFactory
    	AbstractAutoProxyCreator#postProcessBeforeInitialization
    	AbstractAutoProxyCreator#postProcessAfterInitialization
```

### BeanPostProcessor注册

```java
PostProcessorRegistrationDelegate#registerBeanPostProcessors(org.springframework.beans.factory.config.ConfigurableListableBeanFactory, org.springframework.context.support.AbstractApplicationContext){
    String[] postProcessorNames = beanFactory.getBeanNamesForType(BeanPostProcessor.class, true, false);

    // Register BeanPostProcessorChecker that logs an info message when
    // a bean is created during BeanPostProcessor instantiation, i.e. when
    // a bean is not eligible for getting processed by all BeanPostProcessors.
    int beanProcessorTargetCount = beanFactory.getBeanPostProcessorCount() + 1 + postProcessorNames.length;
    beanFactory.addBeanPostProcessor(new BeanPostProcessorChecker(beanFactory, beanProcessorTargetCount));

    //BeanPostProcessor分为三种类型：PriorityOrdered,Ordered,没有实现Ordered接口
    //将三种BeanPostProcessor分别放入三个List中 
    List<BeanPostProcessor> priorityOrderedPostProcessors = new ArrayList<BeanPostProcessor>();
    List<BeanPostProcessor> internalPostProcessors = new ArrayList<BeanPostProcessor>();
    List<String> orderedPostProcessorNames = new ArrayList<String>();
    List<String> nonOrderedPostProcessorNames = new ArrayList<String>();
    for (String ppName : postProcessorNames) {
        if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) {
            BeanPostProcessor pp = beanFactory.getBean(ppName, BeanPostProcessor.class);
            priorityOrderedPostProcessors.add(pp);
            if (pp instanceof MergedBeanDefinitionPostProcessor) {
                internalPostProcessors.add(pp);
            }
        }
        else if (beanFactory.isTypeMatch(ppName, Ordered.class)) {
            orderedPostProcessorNames.add(ppName);
        }
        else {
            nonOrderedPostProcessorNames.add(ppName);
        }
    }
	//Ordered接口的BeanPostProcessor都要先排序，在注册
    // First, register the BeanPostProcessors that implement PriorityOrdered.
    sortPostProcessors(priorityOrderedPostProcessors, beanFactory);
    registerBeanPostProcessors(beanFactory, priorityOrderedPostProcessors);

    // Next, register the BeanPostProcessors that implement Ordered.
    List<BeanPostProcessor> orderedPostProcessors = new ArrayList<BeanPostProcessor>();
    for (String ppName : orderedPostProcessorNames) {
        BeanPostProcessor pp = beanFactory.getBean(ppName, BeanPostProcessor.class);
        orderedPostProcessors.add(pp);
        if (pp instanceof MergedBeanDefinitionPostProcessor) {
            internalPostProcessors.add(pp);
        }
    }
    sortPostProcessors(orderedPostProcessors, beanFactory);
    registerBeanPostProcessors(beanFactory, orderedPostProcessors);

    // Now, register all regular BeanPostProcessors.
    List<BeanPostProcessor> nonOrderedPostProcessors = new ArrayList<BeanPostProcessor>();
    for (String ppName : nonOrderedPostProcessorNames) {
        BeanPostProcessor pp = beanFactory.getBean(ppName, BeanPostProcessor.class);
        nonOrderedPostProcessors.add(pp);
        if (pp instanceof MergedBeanDefinitionPostProcessor) {
            internalPostProcessors.add(pp);
        }
    }
    registerBeanPostProcessors(beanFactory, nonOrderedPostProcessors);

    // Finally, re-register all internal BeanPostProcessors.
    sortPostProcessors(internalPostProcessors, beanFactory);
    registerBeanPostProcessors(beanFactory, internalPostProcessors);

    // Re-register post-processor for detecting inner beans as ApplicationListeners,
    // moving it to the end of the processor chain (for picking up proxies etc).
    beanFactory.addBeanPostProcessor(new ApplicationListenerDetector(applicationContext));    
}
```

### BeanPostProcessor创建流程

```java
AbstractAutowireCapableBeanFactory#initializeBean(java.lang.String, java.lang.Object, org.springframework.beans.factory.support.RootBeanDefinition){
    
    if (System.getSecurityManager() != null) {
        AccessController.doPrivileged(new PrivilegedAction<Object>() {
            @Override
            public Object run() {
                invokeAwareMethods(beanName, bean);
                return null;
            }
        }, getAccessControlContext());
    }
    else {
        //注入BeanNameAware、BeanClassLoaderAware、BeanFactoryAware
        invokeAwareMethods(beanName, bean);
    }

    Object wrappedBean = bean;
    if (mbd == null || !mbd.isSynthetic()) {
        //应用后置处理器postProcessBeforeInitialization()
        wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
    }

    try {
        //执行自定义init方法
        invokeInitMethods(beanName, wrappedBean, mbd);
    }
    catch (Throwable ex) {
        throw new BeanCreationException(
            (mbd != null ? mbd.getResourceDescription() : null),
            beanName, "Invocation of init method failed", ex);
    }
    if (mbd == null || !mbd.isSynthetic()) {
       	//应用后置处理器postProcessAfterInitialization()
        wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
    }
    return wrappedBean;
}
```

## 声明式事务

### @EnableTransactionManagement

* 配置类中开启事务管理器

### PlatformTransactionManager

* 向容器中要注入PlatformTransactionManager组件，该组件需要数据源

  ```java
  @Bean
  public PlatformTransactionManager transactionManager() throws Exception{
      return new DataSourceTransactionManager(dataSource());
  }
  ```

### @Transactional

* 标注在方法或类上，该类或方法使用spring事务控制

## 扩展点

### BeanPostProcessor

* bean后置处理器，bean创建对象，及自定义init()前后进行拦截工作的

  ```java
  public interface BeanPostProcessor {
  	Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException;
  	Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException;
  }
  ```

### BeanFactoryPostProcessor

* 在BeanFactory标准初始化之后调用，来定制和修改BeanFactory的内容

* 所有的BeanDefinition已经保存加载到beanFactory，但是bean的实例还未创建时调用

  ```java
  public interface BeanFactoryPostProcessor {
  	void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException;
  }
  ```

### BeanDefinitionRegistryPostProcessor

* 继承BeanFactoryPostProcessor接口，增加了postProcessBeanDefinitionRegistry()

  ```java
  public interface BeanDefinitionRegistryPostProcessor extends BeanFactoryPostProcessor {
  	void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) throws BeansException;
  }
  ```

* 在所有BeanDefinition将要被加载，bean实例还未创建时调用，但优先于BeanFactoryPostProcessor执行

* 执行顺序方法入口

  ```java
  PostProcessorRegistrationDelegate#invokeBeanFactoryPostProcessors(org.springframework.beans.factory.config.ConfigurableListableBeanFactory, java.util.List<org.springframework.beans.factory.config.BeanFactoryPostProcessor>){
      String[] postProcessorNames =					beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);
          invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry);
      ...
          String[] postProcessorNames =
  				beanFactory.getBeanNamesForType(BeanFactoryPostProcessor.class, true, false);
      invokeBeanFactoryPostProcessors(priorityOrderedPostProcessors, beanFactory);
  }
  ```

### ApplicationListener

* 通过事件机制触发调用

  ```java
  public class MyApplicationListener implements ApplicationListener<ApplicationEvent> {
      @Override
      public void onApplicationEvent(ApplicationEvent event) {
          System.out.println("MyApplicationListener-----" + event);
      }
  }
  //发布事件
  context.publishEvent(new ApplicationEvent(new String("自定义事件")){});
  ```

## SpringMVC

### AbstractAnnotationConfigDispatcherServletInitializer

* 配置spring容器类、springmvc的自容器类、拦截路径

  ```java
  public class WebAppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {
      @Override
      protected Class<?>[] getRootConfigClasses() {
          return new Class[]{RootConfiguration.class};
      }
  
      @Override
      protected Class<?>[] getServletConfigClasses() {
          return new Class[]{WebAppConfiguration.class};
      }
  
      /**
       * /：拦截所有请求（包括静态资源（xx.js,xx.png）），但是不包括*.jsp；
       * /*：拦截所有请求；连*.jsp页面都拦截；jsp页面是tomcat的jsp引擎解析的；
       */
      @Override
      protected String[] getServletMappings() {
          return new String[]{"/"};
      }
  }
  ```

* spring容器扫描出@Controller外的所有注解

  ```java
  @ComponentScan(basePackages = "com.foo",
          excludeFilters = @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = {Controller.class}))
  public class RootConfiguration {
  }
  ```

* springmvc容器只扫描@Controller

  ```java
  @ComponentScan(basePackages = "com.foo",
          includeFilters = @ComponentScan.Filter(classes = {Controller.class}), useDefaultFilters = false)
  @EnableWebMvc
  public class WebAppConfiguration extends WebMvcConfigurerAdapter {
  }
  ```

### @EnableWebMvc

* 在配置类中开启对WebMvc的支持

### WebMvcConfigurer、WebMvcConfigurerAdapter

* 自定义SpringMVC的配置

* WebMvcConfigurerAdapter对WebMvcConfigurer做了空实现

  ```java
  @ComponentScan(basePackages = "com.foo",
          includeFilters = @ComponentScan.Filter(classes = {Controller.class}), useDefaultFilters = false)
  @EnableWebMvc
  public class WebAppConfiguration extends WebMvcConfigurerAdapter {
      @Override
      public void configureViewResolvers(ViewResolverRegistry registry) {
          //默认所有的页面都从 /WEB-INF/ xxx .jsp
          //registry.jsp();
          registry.jsp("/WEB-INF/views/", ".jsp");
      }
      //静态资源访问
      @Override
      public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
          configurer.enable();
      }
  }
  ```

### 加载原理

* 配置在spring-web/4.3.19.RELEASE/spring-web-4.3.19.RELEASE.jar!/META-INF/services/javax.servlet.ServletContainerInitializer中的类会被加载

  ```java
  org.springframework.web.SpringServletContainerInitializer
  ```

* SpringServletContainerInitializer类会实例化WebApplicationInitializer，并调用onStartup()

  ```java
  @Override
  public void onStartup(Set<Class<?>> webAppInitializerClasses, ServletContext servletContext)
      throws ServletException {
  
      List<WebApplicationInitializer> initializers = new LinkedList<WebApplicationInitializer>();
  
      if (webAppInitializerClasses != null) {
          for (Class<?> waiClass : webAppInitializerClasses) {
              // Be defensive: Some servlet containers provide us with invalid classes,
              // no matter what @HandlesTypes says...
              if (!waiClass.isInterface() && !Modifier.isAbstract(waiClass.getModifiers()) &&
                  WebApplicationInitializer.class.isAssignableFrom(waiClass)) {
                  try {
                      initializers.add((WebApplicationInitializer) waiClass.newInstance());
                  }
                  catch (Throwable ex) {
                      throw new ServletException("Failed to instantiate WebApplicationInitializer class", ex);
                  }
              }
          }
      }
      if (initializers.isEmpty()) {
          servletContext.log("No Spring WebApplicationInitializer types detected on classpath");
          return;
      }
  
      servletContext.log(initializers.size() + " Spring WebApplicationInitializers detected on classpath");
      AnnotationAwareOrderComparator.sort(initializers);
      for (WebApplicationInitializer initializer : initializers) {
          initializer.onStartup(servletContext);
      }
  }
  ```


## 内嵌Tomcat容器

* debug需在idea配置命令行这行

```xml
<plugins>
    <plugin>
        <groupId>org.apache.tomcat.maven</groupId>
        <artifactId>tomcat7-maven-plugin</artifactId>
        <version>2.2</version>
        <configuration>
            <path>/</path>
            <port>8080</port>
            <server>tomcat7</server>
        </configuration>
        <executions>
            <execution>
                <phase>package</phase>
                <goals>
                    <goal>run</goal>
                </goals>
            </execution>
        </executions>
    </plugin>
</plugins>
```

