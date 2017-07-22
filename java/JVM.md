

## JVM基本结构

### PC寄存器

* 线程私有
* 指向下一条指令的地址
* native方法时为undefine
* 没有任何OutOfMemoryError

### 方法区

* 线程共享
* 保存JVM加载的类信息、常量、静态变量（JDK6时，String常量存放在方法区，JDK1.7时，移到堆里）
* 分代收集算法中通常和Perm关联在一起
* OutOfMemoryError：无法满足内存分配时

### Java堆
 
* 线程共享
* 对象保存在堆中
* GC的主要工作区域，堆也是分代的
* OutOfMemoryError：没有内存完成对象分配，并且堆无法扩展

### Java栈

* 线程私有
* 栈由栈帧组成，每一个方法对应一个栈帧，帧保存一个方法的局部变量、操作数栈、常量池指针
* OutOfMemryError：JVM动态扩展无法申请到内存时
* StackOverFlowError：线程请求的栈深度大于JVM所允许的深度

### 本地方法栈

JVM使用native方式时需要调用本地方法栈

### 直接内存

直接内存在JVM外，直接向系统申请的内存空间，JavaNIO可以直接使用直接内存



## Java Stack

1. Java栈的操作和线程密切相关，线程执行的基本行为就是函数调用，每次函数调用的数据都是通过Java Stack传递的

2. 先进后出的数据结构，只支持出栈、入栈两种操作

3. 栈中保存的主要内容为栈帧，每一次函数调用，都会有一个对应的栈帧被入栈，每一个函数调用结束，都对有一个栈帧被弹出栈


### 栈帧

* 栈帧包含局部变量表、操作数栈、帧数据区

* 函数每次调用都生成对应的栈帧，栈帧占用栈空间，栈空间不足将导致函数无法进行调用，抛出StackOverflowError

---
	public static void recursion(){
		count++;
		recursion();
	}

	-Xss128K
	deep=1084
	java.lang.StackOverflowError

	-Xss256K
	deep=2744
	java.lang.StackOverflowError

* 函数嵌套调用的深度大小由栈的大小决定，栈空间越大可以支持的调用次数越多

#### 局部变量表

* 保存函数的参数以及局部变量，局部变量表中的变量只在当前函数中有效，函数调用结束后，随着栈帧的销毁而被销毁

* 如果函数的参数或局部变量较多，局部变量表会膨胀，使每一次函数调用占用更多的栈空间，导致函数调用深度减少

---

	public static void recursion(){
		count++;
		recursion();
	}
	-Xss128K
	deep=1096
	java.lang.StackOverflowError

	public static void recursion(long a,long b,long c){
		long e=1,f=2,g=3,h=4,i=5,j=6,k=6,l=7,m=8,n=9,o=10;
		count++;
		recursion(e, f, g);
	}
	-Xss128K
	deep=286
	java.lang.StackOverflowError

* 在相同-Xss的配置下局部变量表的大小直接影响函数调用的深度

---	
	public void localvarGc1() {
		byte[] a = new byte[6 * 1024 * 1024];
		System.gc();
	}
	[GC 7634K->6712K(94208K), 0.0123136 secs]
	[Full GC 6712K->6605K(94208K), 0.0112873 secs]

* byte[]被变量a引用，无法回收空间

---
	public void localvarGc2() {
		byte[] a = new byte[6 * 1024 * 1024];
		a = null;
		System.gc();
	}

	[GC 7634K->568K(94208K), 0.0011350 secs]
	[Full GC 568K->461K(94208K), 0.0112390 secs]

* 在gc前a=null，byte[]失去引用，gc回收byte[]

---

	public void localvarGc3() {
		{
			byte[] a = new byte[6 * 1024 * 1024];
		}
		System.gc();
	}
	[GC 7634K->6728K(94208K), 0.0031776 secs]
	[Full GC 6728K->6605K(94208K), 0.0113971 secs]

* 局部变量a离开作用域，但byte[]仍然被a所引用，a仍然存在于局部变量表中

---

	public void localvarGc4() {
		{
			byte[] a = new byte[6 * 1024 * 1024];
		}
		int c = 10;
		System.gc();
	}
	[GC 7634K->552K(94208K), 0.0010780 secs]
	[Full GC 552K->461K(94208K), 0.0114193 secs]

* a离开作用域后失效，变量c复用a的字（槽位），byte[]没有被任何变量引用

---
	public void localvarGc5() {
		localvarGc1();
		System.gc();
	}
	[GC 7634K->6712K(94208K), 0.0031428 secs]
	[Full GC 6712K->6605K(94208K), 0.0115395 secs]
	[GC 6605K->6605K(94208K), 0.0018007 secs]
	[Full GC 6605K->461K(94208K), 0.0082976 secs]

* localvarGc1()中并没有释放内存，但在localvarGc1()返回后栈帧被销毁， 栈帧中所有局部变量表也随之销毁，byte[]失去引用

#### 操作数栈

用于保存计算的中间结果，同时作为计算过程中变量临时的存储空间

#### 帧数据区

保存访问常量池的指针，方便程序访问常量池

### 栈上分配

* JVM中的一项优化技术，对于线程私有对象，可以将它们打散，分配在栈上，而不是分配在堆上，好处是在函数调用结束后可以自行销毁，不需要gc介入

* 逃逸分析的目的是判断对象的作用域是否可能逃逸出函数体
	
---
	//u可以被任何线程所访问，u是个逃逸对象
	private static User u;
	public static void alloc() {
		u = new User();
		u.id = 5;
		u.name = "geym";
	}
 
	// u以局部变量的形式存在，并且没有被alloc()返回，或存在任何形式的公开，未发生逃逸
	public static void alloc(){
        User u=new User();
        u.id=5;
        u.name="geym";
	}

* p34 逃逸分析例子，无法复现

## 方法区

* 在JDK1.6、1.7中方法区可以理解为永久区（Perm），默认大小64MB

* 如果系统中存在大量动态代理生成的对象，默认的永久区大小可能会导致内存溢出 

---

	public static void main(String[] args) {
		int i = 0;
		try {
			for (i = 0; i < 100000; i++) {
				CglibBean bean = new CglibBean("geym.zbase.ch2.perm" + i, new HashMap());
			}
		} catch (Exception e) {
			System.out.println("total create count:" + i);
			throw e;
		}
	}

	JDK1.6 1.7 -XX:+PrintGCDetails -XX:PermSize=5M -XX:MaxPermSize=5m

	total create count:94
	[GCException in thread "main"  [PSYoungGen: 1182K->64K(46080K)] 
	Heap
	 PSYoungGen      total 52736K, used 0K [0x00000000e0000000, 0x00000000e3500000, 0x0000000100000000)
	  eden space 52224K, 0% used [0x00000000e0000000,0x00000000e0000000,0x00000000e3300000)
	  from space 512K, 0% used [0x00000000e3480000,0x00000000e3480000,0x00000000e3500000)
	  to   space 1024K, 0% used [0x00000000e3300000,0x00000000e3300000,0x00000000e3400000)
	 ParOldGen       total 844288K, used 688K [0x00000000a0000000, 0x00000000d3880000, 0x00000000e0000000)
	  object space 844288K, 0% used [0x00000000a0000000,0x00000000a00ac218,0x00000000d3880000)
	 PSPermGen       total 4096K, used 4096K [0x000000009fc00000, 0x00000000a0000000, 0x00000000a0000000)
	  object space 4096K, 100% used [0x000000009fc00000,0x00000000a0000000,0x00000000a0000000)
	
	Exception: java.lang.OutOfMemoryError thrown from the UncaughtExceptionHandler in thread "main"

* 在产生94个代理对象后，Perm区内存溢出

## 参数配置

	-server		切换server模式

	-Xss128K		线程栈最大空间
	-Xmx10m		最大堆内存
	-Xms10m 		最小堆内存
	-XX:PermSize=5M		最小永久区
	-XX:MaxPermSize=5m	最大永久区
	-XX:MaxMetaspaceSize=5M		JDK1.8使用

	-XX:+PrintGC		打印gc日志

	-XX:+DoEscapeAnalysis	启用逃逸分析
	-XX:+EliminateAllocations	开启标量替换（默认）
  