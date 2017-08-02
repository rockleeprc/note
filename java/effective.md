


### Iitem13 使类和成员的可访问性最小化

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

* 
