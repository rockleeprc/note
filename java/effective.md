
## 第二章 创建和销毁对象

### Item1:考虑用静态工厂方法代替构造器

* 类可以提供一个公有的静态工厂方法,只是一个返回类的实例静态方法

		public static final Boolean TRUE = new Boolean(true);
	 	public static final Boolean FALSE = new Boolean(false);
		public static Boolean valueOf(boolean b) {
				 return (b ? TRUE : FALSE);
		}

* 使用静态工厂方法而不是构造器的优势
	1. 有名称:构造器与类同名,通过参数列表进行区分,静态工厂方法可以定义不同的方法签名,易于阅读,并慎重的选择名称以便突出它们之间的区别
	2. 不用每次调用时都创建一个新的对象:预先创建好对象,或是通过缓存机制避免创建不必要的对象,Boolean.valueOf()从不创建对象
	3. 返回方法返回类型的子类型对象,返回类型可以是私有类型,目的是隐藏实现的具体细节,这种方式适用于基于接口的框架
	4. 创建泛型实例时,代码更加简洁,JDK1.8中已经不存在这种问题了

		
			Map<String, List<String>> map = new HashMap<String, List<String>>();
			public class MapUtils {
	
				public static <K,V> HashMap<K,V> newInstance(){
					return new HashMap<K,V>();
				}
			}

* 静态工厂方法的缺点
	1. 如果类不含公有的或受保护的构造器,就不能子类化,无法使用继承的同时,鼓励使用组合的方式来代替继承
	2. 构造对象的静态工厂方法与其它静态方法实际上没有任何区别,只能通过惯用名称进行区分
		* valueOf,返回的实例于参数具有相同值,意义上是一种类型转换
		* of,valueOf的简洁替代
		* getInstance,返回唯一的实例
		* newInstance,返回的实例每次都时新创建的
		* getType

### Item2：遇到多个构造器参数时要考虑用构建器

* 静态工厂方法和构造器有共同的距现象，不能很好的扩展到大量的可选参数

* 构造器重载，将使构造器变得庞大，参数阅读性低

* JavaBean模式的setter()在构造过程中可能导致对象状态不一致，状态不一致就有可能导致线程安全问题

* Builder模式，不直接生成想要的对象，让客户端利用所有必要的参数调用构造器或静态工厂方法，得到一个Builder对象

		public class Person {
			//require
			private String name;
			private int age;
			//optional
			private double height;
			private double weight;

			//setter/getter略
		
			// 私有化
			private Person(Builder builder) {
				//在Person内进行数据验证，而不是在Builder内
				this.name = builder.name;
				this.age = builder.age;
				this.height = builder.height;
				this.weight = builder.weight;
			}
		
			// 不直接生成对象，客户端使用Builder实例化
			public static class Builder {
				private String name;
				private int age;
				private double height;
				private double weight;
		
				public Builder(String name, int age) {
					super();
					this.name = name;
					this.age = age;
				}
		
				public Person build() {
					return new Person(this);
				}

				//setter/getter略		
			}
		
			@Override
			public String toString() {
				return "Person [name=" + name + ", age=" + age + ", height=" + height + ", weight=" + weight + "]";
			}
		
			public static void main(String[] args) {
				Person p1 = new Person.Builder("Lee", 18).setHeight(183).setWeight(70).build();
				System.out.println(p1);
				Person p2 = new Person.Builder("Lee", 18).setHeight(183).build();
				System.out.println(p2);
				
			}
		}

* 当需要用Builder创建多个对象时，可以使Builder实现一个公共有的接口，客户端把Builder对象传递给Person

* Class.newInstance()总是企图调用类的无参构造器，这个构造器可能根本不存在

* Builder模式比构造器重载更加冗长，只有在有很多参数时才使用

* 类的构造器或者静态工厂中具有多个参数，当大多数参数都是可选的时候，优先考虑使用Builder模式构建对象

### Item3：用私有构造器或者枚举类型强化Singleton属性

* 构造器私有，提供public static final的字段
		
		public class Singleton1 {
			public static final Singleton1 INSTACE = new Singleton1();
		
			private Singleton1() {
			}	
		}

* 构造器私有，提供private static final的字段，使用静态工厂方法获得实例
		
		public class Singleton2 {
			private static final Singleton2 INSTACE = new Singleton2();
		
			private Singleton2() {
			}
		
			public static Singleton2 getInstance() {
				return INSTACE;
			}
		}

* 使用枚举类，无常提供序列化机制，防止多次实例化，阻止反射攻击
		
		public enum Singleton3 {
			INSTANCE;
		}

### Item4：通过私有构造器强化不可实例化的能力

* 工具类不应该为实例化，实例化一个工具类没有任何意义，显示的声明一个私有的构造器，副作用是不能被子类花

### Item5：避免创建不必要的对象

* 最好能重用对象而不是在每次需要的时候创建一个相同功能的新对象，如果对象是不可用的，它就始终可以被重用

* 对于同时提供静态工厂方法和构造器的类，优先使用静态工厂方法（Boolean.valueOf()、new Boolean()）

* 优先考虑使用基本类型而不是包装类，当心无意识的自动装箱，包装类每次使用都会自动创建新的对象 

* 本Item不是暗示创建对象的代价非常昂贵，而是应该尽可能避免创建对象，小对象的创建和回收动作是非常廉价的

* 维护自己的对象池避免创建对象不是一个好做法，正确使用对象池的典型示例就是数据库连接池  

* 区别于保护性拷贝
	* Item5：重用现有对象时，请不要创建新对象
	* Item39：应该创建新对象时，请不要重用现有对象

### Item6：消除过期的对象引用

* 过期引用指也不会被解除的引用

		public Object pop() {
			if (size == 0)
				throw new EmptyStackException();
			// 只改动size的位置，elements[]中被pop的对象不会被gc，引用还存在
			return elements[--size];
		}
	
		public Object pop2() {
			if (size == 0)
				throw new EmptyStackException();
			Object result = elements[--size];
			//数组元素被释放掉后，该元素的引用就应该被手动清空
			elements[size] = null;
			return result;
		}

* 如果一个对象的引用被无意识的保留起来，垃圾回收机制不会回收这个对象，也不会回收这个对象所引用的其它对象
 
* 清空对象引用应该是一种例外，而不是一种规范的行为 

* 只要类自己管理内存，程序员就应该警惕内存泄漏问题 

* 内存泄漏另一个常见来源是缓存 

* 监听器和回调引起的内存的泄漏，一个服务端API，客户端在API中注册回调，却没有显示的取消注册，这个注册将永远存在与服务端 

### Item7：避免使用finalize()

* finalize()是不可预测的，JVM延迟执行finalize()，该方法会导致行为的不稳定，降低性能，一般情况下不是必要，应该避免使用

* Java语言规范不会保证finalize()被及时的执行，而是根本不保证会它会被执行，不应该依赖于finalize()更新重要的持久状态

* 显示的终结方法必须在一个私有域中记录该对象已经不再有效，如果对象在被终结后调用终结方法，将抛出异常（InputStream、OutputStream、java.sql.Connection）,显示终结方法常与try{}finally{}一起使用

* 除非作为安全网或为了终止非关键的本地资源，不要使用finalize()

 


## 第四章 类和接口

### Item13：使类和成员的可访问性最小化

* 设计良好的模块可以对外部其它模块隐藏内部数据和实现细节，把API与实现清晰隔离开，这也是面向对象封装性的体现

* 尽可能地使每个类或者成员不被外加访问（最小访问级别）

* 访问权限
	* private：当前类内部
	* default：包内任何类，默认权限
	* protected：包内任何类，子类
	* public：任何地方都可以访问

* 实现Serializable接口的类可能会被泄漏出定义的API

* 子类覆盖父类中的方法，子类中方法的访问级别不允许低于父类中的访问级别，接口中的所有方法都是public

* public fianl、final引用可变对象，都将放弃对字段的访问限制功能，public static final定于的全局常量在系统中被引用，这是例外情况

* 长度非0的数组总是可变的，类中具有public static fianl的数组，或存在这种数组的getter方法，这几乎总是错误的行为

* 解决publi static fianl 可安全访问的两种方法

		# public数组变private数组，增加一个public的不可变List
		private static final String[] PRIVATE_VALUES = { "A", "B", "C" };
		public static final List<String> VALUES = Collections.unmodifiableList(Arrays.asList(PRIVATE_VALUES));

		# public数组变private数组，返回私有数组的备份
		private static final String[] PRIVATE_VALUES = { "A", "B", "C" };
		public static final String[] values(){
			return PRIVATE_VALUES.clone();
		}

### Item14：在公有类中使用访问方法而非共有域

* 面向对象程序设计中，类应该包含私有字段，公共有访问方法（getter），对于可变的字段，应该包含私有字段和射值方法（setter）

* 如果类可以在它所在的包的外部进行访问，就应该提供访问方法

### Item15：使可变性最小化

* 不可变类是实例不能被修改的类，每个实例中包含的所有信息都必须在创建该实例的时候就提供，并在对象的整个生命周期内固定不变（String、基本类型包装类、BigInteger）

* 类不可变5原则
	1. 不提供任何会修改对象状态的方法
	2. 保证类不会被扩展（final class、private构造器）
	3. 使类中所有字段都是final的（清晰的表明意图）
	4. 使类中所有字段都是private的（防止客户端直接修改字段）
	5. 确保对于任何可变组件的互斥访问（类中包含一个可变对象字段，确保客户端不能直接修改这个对象）

* 不可变对象本质上线程安全的，它们不要求同步，不可变对象可以被自由的共享

* 不可变对象不需要保护性拷贝，也不应该有clone方法（String却有clone方法）

* 不可变类可以使用静态工厂获得，考虑使用静态工厂替代公有构造器

* 不可变类唯一的缺点是，对于每个不同的值都需要一个单独的对象，创建对象的代价可能很高

* 坚决不要为每个getter()编写一个setter()，除非有很好的理由

* 如果类不能被做成不可变的，仍然应该进可能的限制它的可变性，降低对象可以存在的状态数

* 类中的每个字段都应尽可能的是final的

* 不要在构造器或静态工厂之外的地方在提供公有的初始化方法，也不应该提供重新初始化的方法

### Item16：复合先于继承

* 继承是实现代码重用一种方法，在包内的继承非常安全，跨越包边界的继承非常危险

* 继承打破了封装性，子类依赖于超类中的特定实现细节，超类发生改变，子类可能遭到破坏（继承HashSet统计元素个数）

* 如果父类在新版本中增加了一个新的方法，该方法与子类拥有完全相同的方法签名和返回类型，子类将overriding超类中的方法

* 如果父类在新版本中增加了一个新的方法，该方法与子类拥有完全相同的方法签名和不同的返回类型，编译将无法通过

* 不使用继承，而使用组合，在新类中包含一个私有字段，引用现有类的一个实例，新类中的每个方法都可以调用私有字段实例的方法，并返回结果

* 继承一个类时，思考父类API有没有缺陷，缺陷会传播到子类的API中，组合可以隐藏API的缺陷

* 只有当子类真正是超类的子类型时，才适合用继承，A和B确定两者是“is-a”关系时，B才应该扩展A（B is a A）

### Item17：要么为继承而设计，并提供文档说明，要么就禁止继承

* 好的API文档应该描述一个给定的方法做了什么工作，而不是描述他如何做到的

* 为了继承而设计的类，唯一的测试方法就是编写子类，3个子类通常足以测试一个可扩展的类

* 可继承类中构造器绝不能调用可被覆盖的方法，无论是直接还是间接

* 如果类被设计为可继承的，实现Cloneable、Serializable接口都不是好主意

### Item18：接口优于抽象类

* 现有的类可以很容易被更新，以实现新的接口，但无法通过更新现有类来扩展新的抽象类

* 接口允许构造非层次结构的类型框架，继承会导致层次结构臃肿

		class A{}
		calss B extends A{}
		class C extends B{}

		interface A{}
		interface B{}
		interface C extends A,B{}

* 对每个接口提供一个抽象类实现，把抽象类和接口的优点结合，接口仍然定义类型，但抽象类接管了所有与接口实现相关的工作（Collections Framework 为每个重要的接口提供了一个AbstractInterface实现）

* 为抽象类增加新方法要比为接口容易的多

*　接口一旦被公开，并且被广泛实现，再想该边这个接口几乎是不可能的

* 接口通常是定义允许多实现的最佳方式，但当演变的容易性比灵活性和功能更为重要时，应该使用抽象类

* 如果公开了一个重要的接口，就应该坚决的考虑同时提供该接口的抽象类

### Item19：接口只用于定义类型

* 常量接口是对接口的不良使用（java.io.ObjectStreamConstants），接口应该只被用来定义标准

### Item20：类层次优于标签类


### Item21：用函数表示策略

* 面向对象中实现策略模式,先声明一个策略接口,并为每种具体策略声明一个实现类

* 具体的策略实现往往使用匿名内部类,,匿名内部类将会在每次执行调用的时候创建新的实例

		String[] array = { "1234", "1", "123", "12", "12345" };
		Arrays.sort(array, new Comparator<String>() {

			@Override
			public int compare(String o1, String o2) {
				// 升序 return o1.length() - o2.length();
				// 降序
				return o2.length() - o1.length();
			}
		});

* 具体策略需要被重用时,使用私有静态内部类,对外提供一个static final实例,这种方法在具体策略上可以实现多个接口，比如同时实现Comparator、Serializable

		public class ComparatorHost {

			public static final Comparator<String> STRING_LENGTH_COMPARATOR = new StrLen();

			private static class StrLen implements Comparator<String>,Serializable {
				@Override
				public int compare(String o1, String o2) {
					return o1.length() - o2.length();
				}
			}
		}

### Item22：优先考虑静态成员类

* 嵌套内部类存在的目的应该只是为它的外围类提供服务

* 静态内部类
	* 静态内部类常见用法是作为外部类的辅助类,仅当于外部类一起使用时才有意义
	* 如果内部类不要求访问外部类实例,就要始终使用静态内部类,使内部类成为外部类的静态成员
	* private static内部类的常见用法是代表外部类对象的组件,例如,Map.Entry就是静态内部类,Entry里的getKey(),getValue()都不需要访问Map

* 非静态内部类
	* 非静态内部类的每个实例都隐含着与于外部类一个实例相关联,必须先创建外部类,才能获取到非静态内部类
	* 非静态内部类的常见用法是Adapter,例如Map,Set,List接口通常使用非静态内部类实现Iterator

* 匿名内部类
	* 没有名字,不是外部类的成员
	* 不能执行instanceof测试
	* 不能实现接口,扩展父类
	* 不能调用任何类成员
	* 过长的匿名内部类影响程序可读性
	* 常见用法
		1. 创建函数对象,如new Comparator<String>()
		2. 创建过程对象,如new Runnable()
		3. 静态工厂方法内部,实例化对象作为返回

* 局部内部类

