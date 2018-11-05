
# 资源访问

## 1、Resource接口

Spring通过Resource接口提供访问底层资源的能力，Spring针对不同资源访问，提供了不同实现类

### 实现类

	ByteArrayRecource：二进制
	ClassPathResource：类路径下资源
		Resource res = new ClassPathResource("ioc/applicationContext-IoC.xml");
	FileSystemResource：文件系统资源
		Resource res = new FileSystemResource("H:\\hadoop.txt");
		InputStream is = res.getInputStream();
	InputStreamResource：对应InputStrea资源
	ServletContextResource：访问Web容器上下文资源
	UrlResource：通过URL访问网络资源

对资源加载进行编码处理：

	Resource res = new FileSystemResource("H:\\hadoop.txt");
	EncodedResource encode = new EncodedResource(res, "GBK");
	String content = FileCopyUtils.copyToString(encode.getReader());

## 2、ResourceLoader接口

使用表达式通过资源加载器加载资源，隐藏底层访问资源的细节

	classpath:
	file:
	http://
	ftp://
	无 根据ApplictionContext具体实现类采用对应的Resource实现类

### ResourceLoader、ResourcePatternResolver接口

ResourcePatternResolver扩展子ResourceLoader，增加Ant风格的资源路径描述

	PathMatchingResourcePatternResolver：Spring提供的标准实现类
	ResourcePatternResolver loader = new PathMatchingResourcePatternResolver();
	Resource resource = loader.getResource("classpath:ioc/applicationContext-IoC.xml")


# 配置文件在容器中的结构

## 1、BeanDefinition接口

BeanDefinition是对Spring配置文件<bean\>标签的抽象，Spring将配置信息转换为容器内部表示的对象，将BeanDefinition注册到BeanDefinitionRegistry中

BeanDefinitionRegistry中的BeanDefinition只在容器启动时、刷新时、重启时加载

### 实现类

	RootBeanDefinition：对应<bean>标签中的父<bean>，没有父<bean>的<bean>就是用RootBeanDefinition
	ChildBeanDefinition：对应<bean>标签的中的子<bean>
	AbstractBeanDefinition：对RootBeanDefinition和ChildBeanDefinition共同信息进行抽象

## 2、InstantiationStrategy接口

InstantiationStrategy负责根据BeanDefinition创建Bean实例，这是一个策略接口，方便采用不同的方式创建Bean实例

### 实现类
	SimpleInstantiationStrategy：通过构造器、带参数的构造器、工厂方法创建Bean实例
	CglibSubclassingInstantiationStrategy：使用CGLib动态生成子类，使用子类创建Bean实例

## 3、BeanWrapper接口

InstantiationStrategy仅负责实例化Bean的操作，并不对Bean进行任何填充，Bean实例化完成后会被BeanWrapper包装起来，进行填充

### 实现类
	BeanWrapperImpl：
		1、包裹Bean的实例
		2、用于设置Bean属性的属性编辑器

# 属性编辑器

将配置文件中的文本值转换成Bean属性能识别的值，属性编辑器的功能就是一个类型转换器

## 1、自定义属性编辑器

继承PropertyEditorSupport，重写setAsText(String text)

	public class CarEditor extends PropertyEditorSupport {
		@Override
		public void setAsText(String text) throws IllegalArgumentException {
			if (text == null || text.indexOf(",") == -1)
				throw new IllegalArgumentException("format error");
			String[] split = text.split(",");
			Car car = new Car(split[0], split[1], Integer.parseInt(split[2]));
			setValue(car);
		}
	}

继承PropertyEditorSupport implements BeanFactoryPostProcessor，是一个Bean工厂的后处理器

配置文件中注册自定义属性编辑器

	<bean id="boss" class="prc.example.ioc.bean.Boss">
		<property name="car" value="红旗,red,200"></property>
	</bean>
	<!-- 注册自定义属性编辑器 -->
	<bean class="org.springframework.beans.factory.config.CustomEditorConfigurer">
		<property name="customEditors">
			<map>
				<entry key="prc.example.ioc.bean.Car">
					<bean class="prc.example.ioc.definition.CarEditor"></bean>
				</entry>
			</map>
		</property>
	</bean>

## 2、外部属性文件

PropertyPlaceholderConfigurer类提供在配置文件中使用${user}、${password}占位符引用外部文件中定义的内容

PropertyPlaceholderConfigurer implenents BeanFactoryPostProcessor，是一个Bean工厂的后处理器

	<context:property-placeholder location="classpath:jdbc.properties" />

# 容器事件
TODO

# SpringIoC核心类

```java
# 加载资源接口
ResourceLoader
# 资源读取接口，Bean解析过程
BeanDefiniationReader
# Bean源数据信息接口
BeanDefiniation
	RootBeanDefinition
	ChildBeanDefinition
 # IoC父接口
BeanFactory
	AutowireCapableBeanFactory
	HierarchicalBeanFactory
	ListableBeanFactory
# 继承AutowireCapableBeanFactory、HierarchicalBeanFactory、ListableBeanFactory
ConfigurableListableBeanFactory 
# 保存bean实例，BeanFactory默认实现
DefaultListableBeanFactory
```

## AbstractApplicationContext

### refresh()：

* 创建ApplicationContext，或在ApplicationContext建立后刷新

```java
public void refresh() throws BeansException, IllegalStateException {
    synchronized (this.startupShutdownMonitor) {
        // Prepare this context for refreshing.
        prepareRefresh();

        // Tell the subclass to refresh the internal bean factory.
        ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

        // Prepare the bean factory for use in this context.
        prepareBeanFactory(beanFactory);

        try {
            // Allows post-processing of the bean factory in context subclasses.
            postProcessBeanFactory(beanFactory);

            // Invoke factory processors registered as beans in the context.
            invokeBeanFactoryPostProcessors(beanFactory);

            // Register bean processors that intercept bean creation.
            registerBeanPostProcessors(beanFactory);

            // Initialize message source for this context.
            initMessageSource();

            // Initialize event multicaster for this context.
            initApplicationEventMulticaster();

            // Initialize other special beans in specific context subclasses.
            onRefresh();

            // Check for listener beans and register them.
            registerListeners();

            // Instantiate all remaining (non-lazy-init) singletons.
            finishBeanFactoryInitialization(beanFactory);

            // Last step: publish corresponding event.
            finishRefresh();
        }

        catch (BeansException ex) {
            if (logger.isWarnEnabled()) {
                logger.warn("Exception encountered during context initialization - " +
                            "cancelling refresh attempt: " + ex);
            }

            // Destroy already created singletons to avoid dangling resources.
            destroyBeans();

            // Reset 'active' flag.
            cancelRefresh(ex);

            // Propagate exception to caller.
            throw ex;
        }

        finally {
            // Reset common introspection caches in Spring's core, since we
            // might not ever need metadata for singleton beans anymore...
            resetCommonCaches();
        }
    }
}
```

