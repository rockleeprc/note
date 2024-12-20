
## SpringAOP：AOP基本概念

### AOP思想起源
AOP是软件开发思想发展到一定阶段后的产物，从OOP滋生而来，但不是为了替代OOP，是对OOP的一个补充。

其实编程语言发展的终极目标是能以更自然的方式模拟这个世界，对于“思想”这种东西向来只能意会，不是不能言传，而是无法言传，其原因还是只有你觉到、悟到后才能得到。

### AOP中的一些术语
AOP中存在大量的术语，并且这些术语在代码中大量应用，还是应该对这些术语加以重视，很可能你学不会AOP的原因就是对于这些基本概念比较模糊，毕竟代码本身还是比较廉价的。

#### Joinpoint（连接点）
一个类或一段程序执行的某个特定的位置被称作Joinpoint，在SpringAOP中这个特定的位置就是方法，划重点了，**Joinpoint在SpringAOP中表示一个类的方法**，所有的增强逻辑都是针对类中的方法完成的（Introduction特殊）。

#### Pointcut（切点）
一组特定的Joinpoint被称作Pointcut，这里有两个概念，第一是“一组”，也就意味着不是一个Joinpoint（不是一个方法），第二是“特定”，有过滤的含义，可以关注需要的Joinpoint而不是类中所有Joinpoint（一定要对Joinpoint这个单词有感觉，马上能反映出在Spring中就是方法）

#### Advice（增强）
作用在Joinpoint上的一段代码被称作Advice（把Advice理解为“通知”不是很恰当），这段代码具有“方向”性，可以在Joinpoint前、后、发生异常后执行，所以脱离Joinpoint单独讨论Advice意义不大，Advice方向性会在后面会详细介绍。

#### Target（目标对象）
在Java中方法是不能脱离对象存在的，Target就是Joinpoint所在的类。

#### Introduction（不知道怎么翻译）
这是一种特殊的Advice，为Target添加一些属性和方法，Advice作用在Joinpoint，Introduction作用在Target。

#### Weaving（织入）
将Advice应用在Joinpoint上的过程成为Weaving，Weaving关注的是过程本身，而不是结果。

AOP织入方式有三种，`编译期织入、装载期织入、代理织入`，其中编译期、装载期需要特殊的编译器、装载器支持，Spring采用的是代理织入。

#### Proxy（代理）
一个Target被Advice后产生的类叫做Proxy

#### Aspect（切面）
将Joinpoint、Advice组合在一起的地方被称作Aspect，SpringAOP就是把Joinpoint、Advice组合在一起的框架。

与AOP相关的基本概念就介绍完了，再次的重申不要觉得这些概念就是几个名词，眼熟就可以了，掌握这些概念对理解SpringAOP非常重要。


## SpringAOP：浅析代理
### 代理模式
代理模式能够给某个对象提供一个代理对象，并由代理对象控制对原对象的引用，控制引用更多的体现在控制对原对象的访问上。
代理在实现方式上有两种，通过一个小栗子演示一下这两种方式，先定义一个`Target`接口，其实现类是`TargetImpl`。
```java
public interface Target {
    String echo(String name);
}
public class TargetImpl implements Target {
    @Override
    public String echo(String message) {
        return message;
    }
}
```
面向对象三要素`封装、继承、多态`，这第一种方式就是利用继承实现代理，思路是继承TargetImpl，重写echo()，在重写的echo()中调用父类的echo()
```java
public class ExtendProxy extends TargetImpl {
    @Override
    public String echo(String name) {
        System.out.println("继承方式实现代理");
        return super.echo(name);
    }
}
```
第二中方式利用类的组合实现代理，思路是代理类实现Target接口，同时代理类持有TargetImpl实例的引用，GOF的代理模式也是这个思路
```java
public class ComponentProxy implements Target {
    private Target taret;
    public ComponentProxy(Target taret) {
        this.taret = taret;
    }
    @Override
    public String echo(String name) {
        System.out.println("组合方式实现代理");
        return taret.echo(name);
    }
}
```
面向对象中的`封装、继承、多态`完全体现在里代理中，基础知识还是非常重要的吧，从AOP知识点的完整性考虑，引入了代理，但不会作为设计模深入展开

### JDK代理
JDK在1.3中提供对代理的支持，可以在运行期创建接口的代理实例，相关的核心类是`Proxy、InvocationHandler、Method`，重点是只能针对接口创建代理，没有接口则无法创建代理。
先来看一个JDK代理的大栗子，还是上文中用到的Target、TargetImpl两个类，针对Target接口做代理
```java
Target jdkProxy = (Target) Proxy.newProxyInstance(
	Target.class.getClassLoader(),
	new Class[]{Target.class},
	(proxy, method, params) -> {
		System.out.println("jdk代理");
		return method.invoke(target, params);
	});
```
jdkProxy就是一个代理对象，直接调用echo()方法就会答应出“jdk代理”这段话，拆解一下这7行程序到低执行了什么，先来看一下Proxy.newProxyInstance()的方法签名
```java
public static Object newProxyInstance(ClassLoader loader,
										Class<?>[] interfaces,
										InvocationHandler h)
```
ClassLoader：一个类加载器，程序中用的是Target类的加载器（对类加载器不展开说明）
Class：目标接口的Class实例，这里传入的是Target
InvocationHandler：这是一个接口，其中只有一个invoke()，重点在这个方法的三个参数上
```java
public interface InvocationHandler {
    public Object invoke(Object proxy, Method method, Object[] args)
        throws Throwable;
}
```
Object：代理之后生成的代理对象，就是程序中的`jdkProxy`实例
Method：目标类中的方法，在Target中就是`echo()`
Object[]：方法参数，参数可能是多个，所以是个数组，在Target中就是`echo(String message)`中的String类型的message

代理逻辑真正发生的地方就在`InvocationHandler.invoke()`中，在这个方法中调用了`method.invoke(target, params)`，执行了目标方法，看上去`Proxy.newProxyInstance()`只不过是针对这个InvocationHandler封装的一个调用，有没有觉得很神奇的事情是用调用`Proxy.newProxyInstance()`就能产生一个代理对象，这个代理对象是如何产生的呢？是采用继承方式实现代理，还是采用组合方式实现代理呢？如果你能想到这些，其实就做了知其然，也想知其所以然。JDK采用了组合的方法（回顾一下之前的组合实现代理），JDK会在内存中生成一个代理类，样子大概如下
```java
public final class $Proxy extends Proxy implements Target {
	private static Method echoMethod;
	static {
		echoMethod = Class.forName("com.foo.proxy.TargetImpl").getMethod("echo", Class.forName("java.lang.String"));
	}
	
	public final String echo(String message) throws  RuntimeException {
		return (String)super.h.invoke(this, echoMethod, new Object[]{message});
	}
}
```
继承了Proxy类，实现了目标接口，既然继承了Proxy就不能在继承TargetImple，Java是但继承的，所以采用了组合方式实现代理，这个`Proxy.newProxyInstance()`返回的就是这个`$Proxy`实例，为了减少不必要的IO操作，生成个$Proxy的类都是在内存中通过JVM提供的native方法完成的

### CGLib代理
JDK代理有个限制是必须针对接口代理，CGBlib可以对没有实现接口的类生成代理对象，举个栗子，看看怎么使用CGLib生成代理
```java
Enhancer enhancer = new Enhancer();
enhancer.setSuperclass(TargetImpl.class);
enhancer.setCallback((MethodInterceptor) (obj, method, args, proxy) -> {
	System.out.println("cglib代理");
	return proxy.invokeSuper(obj, args);
});
Target cglibProxy = (Target) enhancer.create();
```
看到CGLib API是不是觉得与JDK API操作思路如出一辙，核心就是MethodInterceptor接口和intercept()
```java
public interface MethodInterceptor extends Callback {
    Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable;
}
```
Object：目标类的实例，程序中指TargetImpl实例
Method：目标类的方法，程序中指TargetImpl中的echo()
Object[]：目标类方法的参数列表
MethodProxy：生成代理的实例

### 抛砖引玉
* 代理到底代理的是什么？
* AOP和代理有什么关系？
* Joinpoint在代理中是怎么体现的？
* Advice在代理中意味着哪些代码？

## SpringAOP：详解Advice
在《SpringAOP：AOP基本概念》中说到Advice是一段代码，并且带有方向性。Rod Johnson提出的在编程时应该面向接口开发，把公共的标准抽象为接口，与具体实现类解耦，SpringAOP也是这么做的，所有的Advice都是接口，Advice中的逻辑需要自己实现，有了之前在《SpringAOP：浅析代理》中的基础，Advice就容易很多，重点应该放在Advice接口上，本篇文章将详细介绍与Advice相关的知识点。

以在《SpringAOP：浅析代理》中出现的Target、TargetImpl为基础，对它们应用Advice，说明每种Advice如何使用
```java
public interface Target {
    String echo(String name);
}
public class TargetImpl implements Target {
    @Override
    public String echo(String message) {
        return message;
    }
}
```

### 前置Advice
前置Advice接口由`org.springframework.aop.MethodBeforeAdvice`提供，这个接口只有一个before()，表示在目标方法执行前被调用，BeforeAdvice是一个空接口，没有任何的方法，留作以后的扩展使用，这种设计是不是很解耦（面向对象的解耦意味着类、接口的增加）
```java
public interface MethodBeforeAdvice extends BeforeAdvice {
	/**
	 * @param method  目标类方法，Target中的所有方法
	 * @param args 目标类方法参数，Target每个方法对应的参数
	 * @param target 目标类实例，TargetImpl实例
	 * @throws
	 */
	void before(Method method, Object[] args, Object target) throws Throwable;
}
public interface BeforeAdvice extends Advice {}
```
对Target应用前置Advice，需要实现MethodBeforeAdvice，自定义Advice逻辑
```java
public class MyBeforeAdvice implements MethodBeforeAdvice {
	public void before(Method method, Object[] args, Object target) throws Throwable {
		System.out.println("This is MyBeforeAdvice。。。");
	}

}
```
现有有了Target，有了Advice如何将Advice应用到Target呢？Spring提供了`org.springframework.aop.framework.ProxyFactory`为我们提供代理对象，将Adving应用在Target上
```java
	Target target = new TargetImpl();
	MethodBeforeAdvice beforeAdvice = new MyBeforeAdvice();

	ProxyFactory pf = new ProxyFactory();
	pf.setTarget(target);
	pf.addAdvice(beforeAdvice);
	pf.setInterfaces(target.getClass().getInterfaces());
	Target proxy = (Target) pf.getProxy();
```
《SpringAOP：浅析代理》中的JDK、CGLib代理都是自己手动实现的，ProxyFactory对这两种代理实现方式做了整合，`setTarget()`设置目标对象，此处是一个Target实例，`addAdvice()`设置Advice，此处使用的是自定义的MyBeforeAdvice实例，`setInterfaces()`设置目标类的接口，此处使用Target接口，如果设置了这个参数，ProxyFactory将使用JDK代理生成代理对象，如果不设置接口，ProxyFactory将使用CGLib生成代理对象，如果目标类即实现了接口，还想使用CGLib呢？可以使用`setOptimize(true)`，ProxyFactory会忽律`setInterfaces()`设置，使用CGLib

显然硬编码这种方式很不Spring，Spring针对代理提供了`org.springframework.aop.framework.ProxyFactoryBean`用于将代理的创建过程交给IoC容器处理，通过名称可知这是一个FactoryBean
```xml
<!-- 
	target-ref：被代理的对象，也就是target对象实例
	proxyInterfaces：被代理对象的接口，也就是Target接口
	interfaces：proxyInterfaces的别名
	interceptorNames：advice、advisor实现类，具体的横切逻辑实现
	proxyTargetClass：true=对target进行cglib代理，true时忽略proxyInterfaces配置
	optimize：true强制使用cglib
	singletion：返回代理对象是否为单例，默认单例
-->
<bean id="targeProxy" class="org.springframework.aop.framework.ProxyFactoryBean"
		p:proxyInterfaces="com.foo.proxy.Target" 
		p:interceptorNames="myBeforeAdvice"
		p:target-ref="targetImpl" 
		p:proxyTargetClass="true">
```
都什么年代了还是在使用xml配置文件吗？Spring的注解和Springboot的自动装配帮我们做了很多事情，屏蔽了底层很多繁琐的细节，xml这种方式虽然很老土，却可以抽丝剥茧，只关注与知识点相关的具体细节。


### 后置Advice
后置Advice接口由`org.springframework.aop.AfterReturningAdvice`提供，接口内只有`afterReturning()`表示在目标方法被调用后执行，使用上也是定义一个实现类，在afterReturning()中添加自定义增强逻辑
```java
public interface AfterReturningAdvice extends AfterAdvice {
	/**
	 * @param returnValue Target类方法返回结果
	 * @param method  Target类方法
	 * @param args Target类方法参数
	 * @param target Target类实例
	 * @throws
	 */
	void afterReturning(Object returnValue, Method method, Object[] args, Object target) throws Throwable;
}
```

### 环绕Advice
环绕Advice接口由`org.aopalliance.intercept.MethodInterceptor`提供，注意一下包路径，这个唯一一个没有在spring包中的Advice接口，`MethodInvocation`封装了目标方法及方法参数，还包括了目标方法实例
```java
public interface MethodInterceptor extends Interceptor {
	Object invoke(MethodInvocation invocation) throws Throwable;
}
```
举个栗子，环绕Advice中通过`invocation.proceed()`使用反射调用Target对象方法，需要注意的是环绕Advice必须有返回值，Advice也可以嵌套执行，比如Advice(Advice(Target))，相当于对Target对象做包装，返回值意味着交由下一个Advice处理
```java
public class MyAroundAdvice implements MethodInterceptor {
	@Override
	public Object invoke(MethodInvocation invocation) throws Throwable {
		System.out.println("环绕前");
		Object obj = invocation.proceed();
		System.out.println("环绕后");
		return obj;
	}
}
```

### 异常Advice
异常Advice接口由`org.springframework.aop.ThrowsAdvice`提供，表示当方法抛出异常时被调用，这个接口没声明任何方法，相当于一个标记接口
```java
public interface ThrowsAdvice extends AfterAdvice {}
```
这个Advice该怎么使用呢？
第一：方法名称必须叫`afterThrowing()`
第二：方法参数有`Method method, Object[] args, Object target,Exception ex`，了解过前置、后置、环绕Advice后，对这三个参数应该秒懂
第三：方法可选参数是`Method method, Object[] args, Object target`，必须同时出现或同时不出现
第四：方法参数最后一个必须是`Throwable`或其子类
```Java
public class MyThrowsAdvice implements ThrowsAdvice {
	public void afterThrowing(Method method, Object[] args, Object target, Exception ex) {
		System.out.println("方法名称：" + method.getName());
		System.out.println("异常信息：" + ex.getMessage());
	}
}
```
来几个栗子看看有效的方法签名都什么样子：
* `afterThrowing(Method method, Object[] args, Object target)`
* `afterThrowing(Exception ex)`
* `afterThrowing(SQLException ex)`

### Introduction Advice
Introduction Advice由接口`org.springframework.aop.IntroductionInterceptor`接口提供，但这个也是一个标记接口，不过spring为其提供了一个默认的实现类`org.springframework.aop.support.DelegatingIntroductionInterceptor`，自定义Introduction时只需要继承这个类就可以了。
```java
public interface IntroductionInterceptor extends MethodInterceptor, DynamicIntroductionAdvice {}
```
Introduction是一个种特殊的Advice，但它的作用不是将Advice应用在方法的前后，Introduction是为目标类新建方法和属性。
举个栗子，为TargetImpl类中添加一个`print(String message)`，首先需要把`print(String message)`封装在一个接口里，定义一个Print接口，内部只有一个`print(String message)`。
```java
public interface  {
	public String print(String message);
}
```
自定义一个类，继承`DelegatingIntroductionInterceptor`并且实现`Print`接口，在`invoke()`中其实也可以实现环绕Advice
```java
public class MyIntroductionAdvice extends DelegatingIntroductionInterceptor implements Print {
	private static final long serialVersionUID = -8213962377263745715L;

	/**
	 * 通过实现接口方法，在目标类织入新的方法
	 */
	@Override
	public String print(String message) {
		return message;
	}

	@Override
	public Object invoke(MethodInvocation mi) throws Throwable {
		System.out.println(" IntroductionAdvice");
		return super.invoke(mi);
	}
```
在xml配置中有别于之前的配置，Introduction通过目标类创建子类的方式完成代理，所以`proxyTargetClass="true"`是必须的配置，`proxyInterfaces`是要添加`print()`所在的接口，其它的配置与之前的Advic配置并无差异
```xml
<bean id="targeProxy" class="org.springframework.aop.framework.ProxyFactoryBean"
		p:proxyInterfaces="com.foo.proxy.Print" 
		p:interceptorNames="myIntroductionAdvice"
		p:target-ref="targetImpl" 
		p:proxyTargetClass="true">
```
调用上要将Target实例强转为Print实例才能调用到print()，就完成了针对Target对象添加print()的功能（再次感觉到面向对象中的多态无处不在）

### 总结
1. Advice接口定义
`org.springframework.aop.MethodBeforeAdvice`
`org.springframework.aop.AfterReturningAdvice`
`org.aopalliance.intercept.MethodInterceptor`
`org.springframework.aop.ThrowsAdvice`
`org.springframework.aop.support.DelegatingIntroductionInterceptor`
2. ThrowsAdvice中的方法需要自定义，DelegatingIntroductionInterceptor是继承
3. 实现方式有两种：`ProxyFactory`、`ProxyFactoryBean`
4. Introduction的Joinpoint不是方法，而是类

**有没有感觉到代理的配置比较麻烦，没生成一个代理类都需要配置，有什么方式可以简化** 

## SpringAOP：切点表达式
切点表达式是AspectJ语言的核心，也是使用@Aspect定义切面的难点所在，本文将详解介绍切点表达式的定义方式

### @Annotation()
表示标注了某个注解的类中所有的方法，注解需要自定义，比如定义一个`@MyAnnotation`注解，目标对象中被`@MyAnnotation`标注的方法都将被拦截，
```java
@Before("@annotation(org.foo.anno.MyAnnotation)")
public void method(){
	Sytem.out.println("织入逻辑代码");
}
```

### execution()
`execution`是最常用的定义切点表达式方式，完整表达式如下
```java
execution(modifiers-pattern? ret-type-pattern declaring-type-pattern?name-pattern(param-pattern) throws-pattern?)
execution(修饰符? 返回值表达式 方法表达式(参数表达式) 异常表达式?)
```
其中`返回值表达式 方法表达式(参数表达式)`为必填，其它可选，用几个栗子讲解这个表达式

**通过方法签名定义切点：**

* `execution(public * *(..))`：方法的修饰符必须是public，第一个`*`代表返回值类型，第二个`*`代表方法名称，`..`代表任意参数
* `execution(* *By)`：匹配所有以By结尾的方法，第一个`*`代表返回值类型，因为方法修饰符可选，这里没有明确指出修饰符，所以匹配所有修饰符，`*By`表示任意以By结尾的方法

**通过类定义切点：**

* `execution(* org.foo.Target.*(..))`：匹配Target接口的所有方法，第一个`*`代表返回值，第二个`*`代表Target接口中的所有方法
* `execution(* org.foo.Target+.*(..))`：匹配Target接口及其所有实现类的方法

**通过包和类定义切点：**

* `execution(* org.foo.*(..))`：匹配`org.foo`包下的所有类的所有方法，`.*`表示包下的所有类
* `execution(* org.foo..*(..))`：匹配`org.foo`包及其子包下的所有类的所有方法，`..*`表示包、子包下的的所有类，`..`在匹配包和类时必须和`*`一起使用
* `execution(* org..*.*DAO.findBy*(..))`：匹配包前缀为`org`包及其子包下类以DAO结尾，且方法名必须以`findBy`开头的所有方法

**通过方法参数定义切点：** 

* `execution(* echo(String,int))`：匹配echo()的第一个参数为String，第二个参数为int，且返回值且任意类型的方法，方法的参数类型为`java.lang`包中的类，直接使用类名，否则必须为全限定类名，比如`echo(java.uitl.List,int)`
* `execution(* echo(String,*))`：第二个参数为任意类型
* `exection(* echo(String,..))`：第一个参数为String类型，后面参数个数、参数类型任意
* `exection(* echo(Object+))`：参数只有一个，类经为Object及其子类

`*`：任意参数类型，`..`：任意参数类型，且个数不限

### args()
`args()`接收一个类名作为参数，表示目标类方法入参必须是该类名及其子类
* `args(org.foo.Target)`：表示目标类方法中入参是Target类及其子类将匹配，等同于`execution(* *(org.foo.Target+))`，args关注参数

### @args()
与`args()`实现的功能一样，只是参数换成了注解，方法被指定注解标注时将匹配切点
* ``