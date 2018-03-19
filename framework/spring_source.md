
* DefaultListableBeanFactory：ListableBeanFactory extends BeanFactory
* BeanDefinitionRegistry：bean注册管理
* BeanDefinition：bean的实例

* FactoryBean是Spring容器提供的一种可以扩展容器对象实例化逻辑的接口，请不要将其与容器名称BeanFactory相混淆，它本身与其他注册到容器的对象一样，只是一个Bean而已，只不过，这种类型的Bean本身就是生产对象的工厂（Factory）

* Spring容器启动流程：
	* 容器启动阶段(对象管理信息的收集)
		* 加载配置文件：BeanDefinitionReader
		* 分析配置信息
		* 配置信息装配到BeanDefinition：将BeanDefinition注册到BeanDefinitionRegistry
	* Bean实例化阶段(根据请求(getBean())生产bean)
		* 实例化对象
		* 装配依赖
		* 生命周期回调
		* 注册回调接口

* BeanFactoryPostProcessor：
	* 允许我们在容器实例化相应对象之前，对注册到容器的BeanDefinition所保存的信息做相应的修改，在容器启动阶段最后加入一道工序
	* BeanFactoryPostProcessor实现：
		* PropertyPlaceholderConfigurer：XML配置文件中使用占位符
		* PropertyOverrideConfigurer：对容器中配置的property信息进行替换
		* CustomEditorConfigurer：辅助性地将后期会用到的信息注册到容器，对BeanDefinition没有做任何变动，用户自定义PropertyEditor的配置
	* BeanFactory使用BeanFactoryPostProcessor：

			// 声明将被后处理的BeanFactory实例
			ConfigurableListableBeanFactory beanFactory = new XmlBeanFactory(new ClassPathResource("..."));
			// 声明要使用的BeanFactoryPostProcessor 
			PropertyPlaceholderConfigurer propertyPostProcessor = new PropertyPlaceholderConfigurer();
			propertyPostProcessor.setLocation(new ClassPathResource("..."));
			// 执行后处理操作
			propertyPostProcessor.postProcessBeanFactory(beanFactory); 

	* ApplicationContext使用BeanFactoryPostProcessor：

			<bean class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
				<property name="locations">
					<list>
						<value>conf/jdbc.properties</value>
						<value>conf/mail.properties</value>
					</list>
				</property>
			</bean>
	
* bean生命周期（AbstractBeanFactory.getBean()、AbstractAutowireCapableBeanFactory.createBean()）
	* 实例化bean对象
		* BeanWrapper
			* 使用策略模式决定如何初始化bean
			* SimpleInstantiationStrategy：通过反射初始化bean
			* CglibSubclassingInstantiationStrategy：继承自SimpleInstantiationStrategy，通过方法注入初始化bean，默认采用CglibSubclassingInstantiationStrategy，返回BeanWrapper的实例，对bean进行包裹
			* BeanWrapper的实现类BeanWrapperImpl
	* 设置对象属性
		* BeanWrapper继承PropertyAccessor，以统一的方法对对象的属性进行访问
	* 检查Aware相关接口并设置相关依赖
		* Spring容器会检查当前对象实例是否实现了一系列的以Aware命名结尾的接口定义。如果是，则将这些Aware接口定义中规定的依赖注入给当前对象实例。
		* 针对BeanFactory容器类型
			* BeanNameAware
			* BeanClassLoaderAware
			* BeanFactoryAware
		* 针对ApplicationContext类型
			* ResourceLoaderAware
			* ApplicationEventPublisherAware
			* MessageSourceAware
			* ApplicationContextAware
	* BeanPostProcessor前置处理
		* BeanPostProcessor是存在于对象实例化阶段，而BeanFactoryPostProcessor则是存在于容器启动阶段
		* Spring的AOP则更多地使用BeanPostProcessor来为对象生成相应的代理对象
		* BeanPostProcessor是容器提供的对象实例化阶段的强有力的扩展点
			
				public Object postProcessBeforeInitialization(Object object, String beanName)				throws BeansException {
					if(object instanceof PasswordDecodable)
					{
						String encodedPassword = ((PasswordDecodable)object).getEncodedPassword();
						String decodedPassword = decodePassword(encodedPassword);
						((PasswordDecodable)object).setDecodedPassword(decodedPassword);
					}
					return object;
				}

				<bean id="passwordDecodePostProcessor" class="package.name.PasswordDecodePostProcessor">
				<!--如果需要，注入必要的依赖-->
				</bean>

	* 检查是否是InitializingBean以决定是否调用afterPropertiesSet方法
		* InitializingBean是容器内部广泛使用的一个对象生命周期标识接口
		* 在对象实例化过程调用过“ BeanPostProcessor的前置处理”之后，会接着检测当前对象是否实现了InitializingBean接口，如果是，则会调用其afterPropertiesSet()方法进一步调整对象实例的状态。
	* 检查是否配置有自定义的init-method
		* 实现InitializingBean接口侵入较大，xml中配置<bean>init-method即可，或<beans>的default-init-method
	* BeanPostProcessor后置处理
	* 注册必要的Destruction相关回调接口(DisposableBean)
	* 使用bean
	* 使用bean是否实现DisposableBean接口
		* 容器将检查singleton类型的bean实例，看其是否实现了org.springframework.beans.factory.DisposableBean接口
		* 对应的bean定义是否通过<bean>的destroy-method属性指定了自定义的对象销毁方法。
		* 与InitializingBean和init-method用于对象的自定义初始化相对应， DisposableBean和destroy-method为对象提供了执行自定义销毁逻辑的机会。
	* 是否配置有自定义的destory方法

* ApplicationContext实现：
	* FileSystemXmlApplicationContext
	* ClassPathXmlApplicationContext
	* XmlWebApplicationContext

* org.springframework.core.io.Resource包下是资源加载实现
* org.springframework.core.io.ResourceLoader接口是资源查找定位策略的统一抽象
	* DefaultResourceLoader：默认实现
	* ClassPathResource：classpath开头的资源



## IoC容器初始化过程


	ApplicationContext applicationContext = new FileSystemXmlApplicationContext(
				"src/main/resources/ioc/applicationContext-beanFactory.xml");
	Person p = (Person) applicationContext.getBean("person");



org.springframework.context.support.FileSystemXmlApplicationContext.FileSystemXmlApplicationContext(String[], boolean, ApplicationContext)

org.springframework.context.support.AbstractApplicationContext.obtainFreshBeanFactory()

org.springframework.context.support.AbstractRefreshableApplicationContext.refreshBeanFactory()

//构建ApplicationContext
org.springframework.context.support.AbstractRefreshableApplicationContext.createBeanFactory()
//装载bean
org.springframework.context.support.AbstractRefreshableApplicationContext.loadBeanDefinitions(DefaultListableBeanFactory)
//装载bean具体实现
org.springframework.context.support.AbstractXmlApplicationContext.loadBeanDefinitions(DefaultListableBeanFactory)
//读取xml配置文件
org.springframework.beans.factory.xml.XmlBeanDefinitionReader.loadBeanDefinitions(EncodedResource)
org.springframework.beans.factory.xml.XmlBeanDefinitionReader.doLoadBeanDefinitions(InputSource, Resource)