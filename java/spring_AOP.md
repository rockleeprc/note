
## 代理
### JDK动态代理
JDK原生提供的代理，但必须针对接口生成代理对象

定义接口

	public interface IPerson {
		void walk();
		void sayHello(String info);
	}

定义接口实现类

	public class Famer implements IPerson {
		public void walk() {
			System.out.println("The famer is walking on the beach");
		}
		public void sayHello(String info) {
			System.out.println("The famer talk to you about you are a bitch ... "+info);
		}
	}

对接口进行代理

	IPerson famer = new Famer();
	IPerson proxy = (IPerson) Proxy.newProxyInstance(famer.getClass().getClassLoader(), famer.getClass().getInterfaces(), new InvocationHandler() {
		/**
		 * proxy：最终生成的代理对象
		 * method：被代理目标实例的某个具体方法
		 * args：传给被代理实例某个方法的参数数组
		 */
		public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
			String methodName = method.getName();
			Object result = null;
			if ("walk".equals(methodName)) {
				System.out.println("invoke method = " + methodName);
				System.out.println("invoke args = " + Arrays.toString(args));
				result = method.invoke(famer, args);
			}
			if ("sayHello".equals(methodName)) {
				System.out.println("invoke method = " + methodName);
				System.out.println("invoke args = " + Arrays.toString(args));
				result = method.invoke(famer, args);
			}
			return result;
		}
	});
	proxy.walk();
	proxy.sayHello("fack");

### Cglib代理
通过字节码，为目标对象生成子类，并在子类中拦截父类方法，提供代理功能，不依赖接口

	Enhancer enhancer = new Enhancer();
	// 设置要创建子类的类
	enhancer.setSuperclass(Famer.class);
	enhancer.setCallback(new MethodInterceptor() {
		public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy)
				throws Throwable {
			String methodName = method.getName();
			//Famer$$EnhancerByCGLIB$$a9b2a6aa
			System.out.println(obj.getClass().getName());
			Object result = null;
			if ("walk".equals(methodName)) {
				System.out.println("invoke method = " + methodName);
				System.out.println("invoke args = " + Arrays.toString(args));
				result = proxy.invokeSuper(obj, args);
			}
			if ("sayHello".equals(methodName)) {
				System.out.println("invoke method = " + methodName);
				System.out.println("invoke args = " + Arrays.toString(args));
				result = proxy.invokeSuper(obj, args);
			}
			return result;
		}
	});
	// 通过字节码动态创建子类
	Famer famer = (Famer) enhancer.create();
	famer.walk();
	famer.sayHello("fack");


## AOP术语

### Jointpoint，连接点
程序执行的某个特定位置，如类初始化前、初始化后，方法前、方法后

Spring仅支持方法类型的Jointpoint

### Pintcut，切点
一组Jointpoint，Spring通过org.springframework.aop.Pointcut接口进行描述，通过Pointcut表达式描述一组Jointpoint，所以一个Pointcut可以匹配多个Jointpoint

### Advice，增强/通知
对Jointpoint要做的行为，一段程序代码，Spring提供了如BeforeAdvice、AfterReturningAdvice带有方位名称的Advice

### Target，目标对象
实施Advice的对象，被代理的目标类

### Intruduction，引介
一种特殊的Advice，为Target添加一些属性和方法

### Weaving，织入
将Advice应用到Target上的Joinpoint的过程

Spring采用动态代理织入（生成子类），AspectJ采用编译器织入（特殊Java编译器）和装在期织入（特殊的装载器）

### Proxy，代理
被AOP Weaving后的结果类

### Aspect，切面
横切逻辑的定义，有Jointpoint、Advice组成


## 切面


### 切点描述

	# 切点描述接口，isRuntim() true=动态匹配，false=静态匹配
	org.springframework.aop.Pointcut
		# 动态方法匹配，每次调用方法都会进行匹配
		org.springframework.aop.support.annotation.AnnotationMatchingPointcut
		# 静态方法匹配，仅对方法签名匹配，只会匹配一次
		org.springframework.aop.support.DynamicMethodMatcherPointcut
		# 基于注解的切点和切点表达式
		org.springframework.aop.support.StaticMethodMatcherPointcut

		## ProxyFactoryBean
	
### Advice
	org.springframework.aop.MethodBeforeAdvice
	org.springframework.aop.AfterReturningAdvice
	org.aopalliance.intercept.MethodInterceptor
	org.springframework.aop.ThrowsAdvice
	org.springframework.aop.support.DelegatingIntroductionInterceptor

### Advisor
	
	1. Advisor：代表一个advice
	2. PointcutAdvisor：具有切点的切面，包含Advice和Pointcut
		1. org.springframework.aop.support.DefaultPointcutAdvisor
		2. org.springframework.aop.support.NameMatchMethodPointcutAdvisor
		3. org.springframework.aop.support.RegexpMethodPointcutAdvisor
		4. org.springframework.aop.support.StaticMethodMatcherPointcutAdvisor
		5. org.springframework.aop.aspectj.AspectJPointcutAdvisor
	3. IntroductionAdvisor：引介

### 配置
	<!-- 
		target-ref：被代理的对象，也就是target对象
		proxyInterfaces：被代理对象的接口，也就是target接口
		interfaces：proxyInterfaces的别名
		interceptorNames：advice、advisor实现类，具体的横切逻辑实现
		proxyTargetClass：true=对target进行cglib代理，不使用jdk
		optimize：true：强制使用cglib
	 -->
	<bean id="xxx" class="org.springframework.aop.framework.ProxyFactoryBean"
		p:proxyInterfaces="target interface" p:interceptorNames="xxxadvice"
		p:target-ref="target object" p:proxyTargetClass="true" optimize="true"></bean>


## AutoProxyCreator
	
	org.springframework.aop.framework.autoproxy.BeanNameAutoProxyCreator
	org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator