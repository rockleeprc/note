

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

* 逃逸分析的目的是判断对象的作用域是否可能逃逸出函数体

---

	public static class User {
		public int id = 0;
		public String name = "";
	}

	public static void alloc() {
		User u = new User();
		u.id = 5;
		u.name = "geym";
	}

	public static void main(String[] args) throws InterruptedException {
		long b = System.currentTimeMillis();
		for (int i = 0; i < Integer.MAX_VALUE; i++) {
			alloc();
		}
		long e = System.currentTimeMillis();
		System.out.println(e - b);
	}

	-server -Xmx10m -Xms10m -XX:+DoEscapeAnalysis -XX:+PrintGC -XX:-UseTLAB  -XX:+EliminateAllocations

* JDK1.7，关闭逃逸分析和标量替换，User对象总是分配在栈上，无法复现书中P35页的情景

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

## JVM跟踪调试参数配置

### -XX:+PrintGC

	[GC 7634K->608K(94208K), 0.0011779 secs]
	[GC gc前->gc后(堆总大小),gc所花费时间]

* 打印gc日志，每次gc打印一行，堆空间使用7634K，gc后使用量608K，堆空间总大小94208K


### -XX:+PrintGCDetails
	
	[GC [PSYoungGen: 7634K->568K(28672K)] 7634K->568K(94208K), 0.0014397 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
	[Full GC [PSYoungGen: 568K->0K(28672K)] [ParOldGen: 0K->465K(65536K)] 568K->465K(94208K) [PSPermGen: 2541K->2540K(21504K)], 0.0113967 secs] [Times: user=0.01 sys=0.00, real=0.01 secs] 
	Heap
	 PSYoungGen      total 28672K, used 737K [0x00000000e0000000, 0x00000000e2000000, 0x0000000100000000)
	  eden space 24576K, 3% used [0x00000000e0000000,0x00000000e00b85e8,0x00000000e1800000)
	  from space 4096K, 0% used [0x00000000e1800000,0x00000000e1800000,0x00000000e1c00000)
	  to   space 4096K, 0% used [0x00000000e1c00000,0x00000000e1c00000,0x00000000e2000000)
	 ParOldGen       total 65536K, used 465K [0x00000000a0000000, 0x00000000a4000000, 0x00000000e0000000)
	  object space 65536K, 0% used [0x00000000a0000000,0x00000000a0074620,0x00000000a4000000)
	 PSPermGen       total 21504K, used 2547K [0x000000009ae00000, 0x000000009c300000, 0x00000000a0000000)
	  object space 21504K, 11% used [0x000000009ae00000,0x000000009b07cde8,0x000000009c300000)

* 打印的日志比-XX:+PrintGC更详细

* 在JVM推出前会打印堆的详细信息

* Full GC 同时回收新生代、老年代、永久代内存

### -XX:+PrintHeapAtGC(-XX:+PrintGC )
	
	{Heap before GC invocations=1 (full 0):
	 PSYoungGen      total 28672K, used 7143K [0x00000000e0000000, 0x00000000e2000000, 0x0000000100000000)
	  eden space 24576K, 29% used [0x00000000e0000000,0x00000000e06f9c98,0x00000000e1800000)
	  from space 4096K, 0% used [0x00000000e1c00000,0x00000000e1c00000,0x00000000e2000000)
	  to   space 4096K, 0% used [0x00000000e1800000,0x00000000e1800000,0x00000000e1c00000)
	 ParOldGen       total 65536K, used 0K [0x00000000a0000000, 0x00000000a4000000, 0x00000000e0000000)
	  object space 65536K, 0% used [0x00000000a0000000,0x00000000a0000000,0x00000000a4000000)
	 PSPermGen       total 21504K, used 2541K [0x000000009ae00000, 0x000000009c300000, 0x00000000a0000000)
	  object space 21504K, 11% used [0x000000009ae00000,0x000000009b07b610,0x000000009c300000)
	[GC 7143K->584K(94208K), 0.0011442 secs]
	Heap after GC invocations=1 (full 0):
	 PSYoungGen      total 28672K, used 584K [0x00000000e0000000, 0x00000000e2000000, 0x0000000100000000)
	  eden space 24576K, 0% used [0x00000000e0000000,0x00000000e0000000,0x00000000e1800000)
	  from space 4096K, 14% used [0x00000000e1800000,0x00000000e1892020,0x00000000e1c00000)
	  to   space 4096K, 0% used [0x00000000e1c00000,0x00000000e1c00000,0x00000000e2000000)
	 ParOldGen       total 65536K, used 0K [0x00000000a0000000, 0x00000000a4000000, 0x00000000e0000000)
	  object space 65536K, 0% used [0x00000000a0000000,0x00000000a0000000,0x00000000a4000000)
	 PSPermGen       total 21504K, used 2541K [0x000000009ae00000, 0x000000009c300000, 0x00000000a0000000)
	  object space 21504K, 11% used [0x000000009ae00000,0x000000009b07b610,0x000000009c300000)
	}

* 在每次GC前后分别打印堆信息

### -XX:+PrintGCTimeStamps(-XX:+PrintGC )

	0.092: [GC 7634K->608K(94208K), 0.0131102 secs]
	0.105: [Full GC 608K->465K(94208K), 0.0116195 secs]

* 每次gc时打印发生时间，时间为虚拟机启动后的时间偏移量 

### -Xloggc:E:\gc.log (-XX:+PrintGC)

* 打印gc日志到文件

### -verbose:class

	[Opened D:\Java\jdk1.7.0_79\jre\lib\rt.jar]
	[Loaded java.lang.Object from D:\Java\jdk1.7.0_79\jre\lib\rt.jar]
	[Loaded java.io.Serializable from D:\Java\jdk1.7.0_79\jre\lib\rt.jar]
	[Loaded java.lang.Comparable from D:\Java\jdk1.7.0_79\jre\lib\rt.jar]
	[Loaded java.lang.CharSequence from D:\Java\jdk1.7.0_79\jre\lib\rt.jar]
	[Loaded java.lang.String from D:\Java\jdk1.7.0_79\jre\lib\rt.jar]
	[Loaded java.lang.reflect.GenericDeclaration from D:\Java\jdk1.7.0_79\jre\lib\rt.jar]
	[Loaded java.lang.reflect.Type from D:\Java\jdk1.7.0_79\jre\lib\rt.jar]
	[Loaded java.lang.reflect.AnnotatedElement from D:\Java\jdk1.7.0_79\jre\lib\rt.jar]
	[Loaded java.lang.Class from D:\Java\jdk1.7.0_79\jre\lib\rt.jar]
	...

* 跟踪类的加载和卸载

* 单独跟踪加载/卸载信息
	* -XX:+TraceClassLoading
	* -XX:+TraceClassUnloading

### -XX:+PrintVMOptions(-XX:+TraceClassLoading -XX:+TraceClassUnloading )

	VM option '+TraceClassLoading'
	VM option '+TraceClassUnloading'
	VM option '+PrintVMOptions'

* 打印JVM接受到的额命令

###  -XX:+PrintCommandLineFlags

	-XX:InitialHeapSize=100631424 -XX:MaxHeapSize=1610102784 -XX:+PrintCommandLineFlags -XX:+TraceClassLoading -XX:+TraceClassUnloading -XX:+UseCompressedOops -XX:-UseLargePagesIndividualAllocation -XX:+UseParallelGC 

* 打印JVM显示和隐含的参数

## 堆参数配置

### 初始堆和最大堆

	# 初始堆内存大小
	-Xms5m
	# 最大堆内存大小 
	-Xmx20m 

* JVM会尽可能维持在Xms空间范围内运行，初始堆空间耗尽，JVM对堆空间进行扩展，上限是Xmx

* 将-Xms与-Xmx设置相等，好处是可以减少程序运行时进行垃圾回收的次数，从而提高性能

### 新生代

* -XX:SurvivorRatio，设置eden与survivor的比例

* -XX:NewRatio，设置老年代与新生代的比例

* 新生代设置的较大，会减小老年代的空间，老年代空间减小后对gc行为影响很大

* 新生代大小一般设置为堆空间的1/3-1/4左右

* 新生代基本的分配策略：尽可能将对象预留在新生代，减少老年代GC的次数

---

	byte[] b = null;
	for (int i = 0; i < 10; i++) {
		b = new byte[1 * 1024 * 1024];
	}

* 分配10MB数组，通过新生代配置，测试JVM内存使用情况

---

	-Xmx20m -Xms20m -Xmn2m   -XX:SurvivorRatio=2 -XX:+PrintGCDetails

	Heap
	 PSYoungGen      total 1536K, used 674K [0x00000000ffe00000, 0x0000000100000000, 0x0000000100000000)
	  eden space 1024K, 65% used [0x00000000ffe00000,0x00000000ffea8818,0x00000000fff00000)
	  from space 512K, 0% used [0x00000000fff80000,0x00000000fff80000,0x0000000100000000)
	  to   space 512K, 0% used [0x00000000fff00000,0x00000000fff00000,0x00000000fff80000)
	 ParOldGen       total 18432K, used 10240K [0x00000000fec00000, 0x00000000ffe00000, 0x00000000ffe00000)
	  object space 18432K, 55% used [0x00000000fec00000,0x00000000ff6000a0,0x00000000ffe00000)
	 PSPermGen       total 21504K, used 2546K [0x00000000f9a00000, 0x00000000faf00000, 0x00000000fec00000)
	  object space 21504K, 11% used [0x00000000f9a00000,0x00000000f9c7cbd8,0x00000000faf00000)

* -Xmn2m：新生代配置为2m

* -XX:SurvivorRatio=eden/from=eden/to

* total 1536K，1024(eden ) + 512(form/to)

* -Xmn2m = 1024k(eden) + 512k(from) + 512k(to)

* 对象进入老年代，增加GC发生的几率

---
	
	-Xmx20m -Xms20m -Xmn7m   -XX:SurvivorRatio=2 -XX:+PrintGCDetails
	
	[GC [PSYoungGen: 3785K->1512K(5632K)] 3785K->1528K(18944K), 0.0012047 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
	[GC [PSYoungGen: 4673K->1528K(5632K)] 4689K->1552K(18944K), 0.0016908 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
	[GC [PSYoungGen: 4631K->1528K(5632K)] 4655K->1560K(18944K), 0.0007706 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
	Heap
	 PSYoungGen      total 5632K, used 2634K [0x00000000ff900000, 0x0000000100000000, 0x0000000100000000)
	  eden space 4096K, 27% used [0x00000000ff900000,0x00000000ffa14820,0x00000000ffd00000)
	  from space 1536K, 99% used [0x00000000ffd00000,0x00000000ffe7e020,0x00000000ffe80000)
	  to   space 1536K, 0% used [0x00000000ffe80000,0x00000000ffe80000,0x0000000100000000)
	 ParOldGen       total 13312K, used 32K [0x00000000fec00000, 0x00000000ff900000, 0x00000000ff900000)
	  object space 13312K, 0% used [0x00000000fec00000,0x00000000fec08000,0x00000000ff900000)
	 PSPermGen       total 21504K, used 2546K [0x00000000f9a00000, 0x00000000faf00000, 0x00000000fec00000)
	  object space 21504K, 11% used [0x00000000f9a00000,0x00000000f9c7cbd8,0x00000000faf00000)

* 程序中每申请一次空间，都会废弃上次申请的内存，新生代中有效的回收了这部分失效的内存

* 没有任何对象进入到老年代

* 新生代区域有足够的空间，所有数组都会被分配到新生代中

---

	-Xmx20m -Xms20m -Xmn15m   -XX:SurvivorRatio=8 -XX:+PrintGCDetails
	
	Heap
	 PSYoungGen      total 13824K, used 11525K [0x00000000ff100000, 0x0000000100000000, 0x0000000100000000)
	  eden space 12288K, 93% used [0x00000000ff100000,0x00000000ffc41738,0x00000000ffd00000)
	  from space 1536K, 0% used [0x00000000ffe80000,0x00000000ffe80000,0x0000000100000000)
	  to   space 1536K, 0% used [0x00000000ffd00000,0x00000000ffd00000,0x00000000ffe80000)
	 ParOldGen       total 5120K, used 0K [0x00000000fea00000, 0x00000000fef00000, 0x00000000ff100000)
	  object space 5120K, 0% used [0x00000000fea00000,0x00000000fea00000,0x00000000fef00000)
	 PSPermGen       total 21504K, used 2546K [0x00000000f9800000, 0x00000000fad00000, 0x00000000fea00000)
	  object space 21504K, 11% used [0x00000000f9800000,0x00000000f9a7cbd8,0x00000000fad00000)

* 新生代total 13824K，满足程序10MB的数组分配，所有分配都在eden区完成，没有触发GC，from/to使用率为0

---
	
	-Xmx20m -Xms20m  -XX:NewRatio=2 -XX:+PrintGCDetails
	
	[GC [PSYoungGen: 5954K->488K(6656K)] 5954K->1536K(20480K), 0.0014987 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
	Heap
	 PSYoungGen      total 6656K, used 5800K [0x00000000ff900000, 0x0000000100000000, 0x0000000100000000)
	  eden space 6144K, 86% used [0x00000000ff900000,0x00000000ffe300c8,0x00000000fff00000)
	  from space 512K, 95% used [0x00000000fff00000,0x00000000fff7a020,0x00000000fff80000)
	  to   space 512K, 0% used [0x00000000fff80000,0x00000000fff80000,0x0000000100000000)
	 ParOldGen       total 13824K, used 1048K [0x00000000feb80000, 0x00000000ff900000, 0x00000000ff900000)
	  object space 13824K, 7% used [0x00000000feb80000,0x00000000fec86010,0x00000000ff900000)
	 PSPermGen       total 21504K, used 2546K [0x00000000f9980000, 0x00000000fae80000, 0x00000000feb80000)
	  object space 21504K, 11% used [0x00000000f9980000,0x00000000f9bfcbd8,0x00000000fae80000)


* -XX:NewRatio=老年代/新生代，设置新生代和老年代的比例

*  PSYoungGen + ParOldGen = -Xms20m

* 新生代GC时有1MB数组存活，from/to不足1MB，需要老年代进行空间担保，有1MB数组进入老年代

### 导出堆信息

    public static void main(String[] args) {
        Vector v=new Vector();
        for(int i=0;i<25;i++)
            v.add(new byte[1*1024*1024]);
    }

	-Xmx20m -Xms5m -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=d:/a.dump


## 直接内存

	# 设置最大直接内存
	-XX:MaxDirectMemorySize=10m

* 直接内存默认的大小等于堆内存空间

* 申请直接需要花费比申请堆内存更多地时间

* 适用于申请次数少，访问较频繁的场合，如果内存空间本身需要频繁申请，并不适合直接内存。


## JVM工作模式

* Client，启动快，优化少

* Server，在启动时尝试收集更多的系统性能西西你，使用复杂的优化算法对程序进行优化，启动时需要更长时间

## 参数配置总结

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
  
	-XX:+PrintReferenceGC	跟踪软引用、若引用、虚引用、Finallize队列


## Linux下性能监控工具

### top

* 显示系统中各个进程的资源占用情况

---
    $ top

    top - 22:32:11 up 12 days, 21:58,  1 user,  load average: 1.70, 1.98, 2.26
    Tasks: 231 total,   1 running, 229 sleeping,   0 stopped,   1 zombie
    %Cpu(s): 22.1 us,  2.3 sy,  0.0 ni, 75.3 id,  0.2 wa,  0.0 hi,  0.1 si,  0.0 st
    KiB Mem :  8054928 total,  1903624 free,  4306368 used,  1844936 buff/cache
    KiB Swap: 15624188 total, 14724060 free,   900128 used.  2813424 avail Mem

     PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND                                  
    31761 mint      20   0 3874232 532416  93340 S  78.4  6.6 809:55.00 plugin-containe                          
    5452 mint      20   0 1858528 204900  40508 S   4.7  2.5 118:41.81 cinnamon      

### vmstat
* 查看内存,交互分区,I/O操作,上下文切换,时钟中断,CPU使用情况
---

    # 采样一次,共计3次
    $ vmstat 1 3
    procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
     r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
     1  0 899464 1749512 163668 1813516    1    5    33    35    3   60 55  6 39  1  0
     0  0 899464 1748628 163668 1813548    0    0     0     0 1131 3662 23  1 76  0  0
     2  0 899464 1748132 163668 1813548    0    0     0     0 1135 3650 23  2 75  1  0



### iostat
* 详细I/O信息

---
    $ sudo apt install sysstat

    # 输出信息每1秒采样一次,合计采样2次
    $ iostat 1 2
    # -d 输出磁盘使用情况,不显示cup使用情况
    $ iostat -d 1 2
    # 更多统计信息
    $ iostat -x 1 2

## JDK监控工具


### jstat
* 用于观察Java应用程序运行时的相关工具

---
    # 与GC相关的堆信息
    $ jstat -gc 1938
     S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   CCSU   YGC     YGCT    FGC    FGCT     GCT   
    20992.0 20992.0  0.0   6490.1 456192.0 107415.9  70656.0    18463.5   32432.0 31617.3 4016.0 3834.0      9    0.208   1      0.074    0.282

    # 与GC相关的内存信息,和各个代的最大/小值
    $ jstat -gccapacity 1938
     NGCMN    NGCMX     NGC     S0C   S1C       EC      OGCMN      OGCMX       OGC         OC       MCMN     MCMX      MC     CCSMN    CCSMX     CCSC    YGC    FGC
     41984.0 671744.0 671232.0 21504.0 22016.0 626688.0    84992.0  1343488.0    70656.0    70656.0      0.0 1077248.0  32432.0      0.0 1048576.0   4016.0     15     1

     # 最近一个GC原因,及当前GC原因
     $ jstat -gccause 1938
     S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT    LGCC                 GCC                 
     0.00   1.68  41.59  35.06  97.60  95.47     17    0.229     1    0.074    0.304 Allocation Failure   No GC               

     # 新生代信息
     $ jstat -gcnew 1938
      S0C    S1C    S0U    S1U   TT MTT  DSS      EC       EU     YGC     YGCT  
      512.0 17920.0  320.0    0.0  8  15 17920.0 493568.0 344500.5     20    0.237

      # 老年代信息
      $ jstat -gcold 1938
      MC       MU      CCSC     CCSU       OC          OU       YGC    FGC    FGCT     GCT   
      32432.0  31655.2   4016.0   3834.0     70656.0     24770.9     21     1    0.074    0.313

      # GC回收相关信息
      $ jstat -gcutil 1938
      S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT   
      0.00   2.16  16.01  35.07  97.65  95.47     25    0.245     1    0.074    0.319

### jinfo
* 查看正在运行Java应用程序的扩展参数
---

    # 显示是否打印GC日志信息
    $ jinfo -flag PrintGCDetails 1938
    -XX:-PrintGCDetails

### jmap

* 生成Java程序的堆Dump文件,查看堆内存对象实例的统计信息


    # 生成1938进程的Java程序对象统计信息
    $ jmap -histo 1938
    $ jmap -histo 1938 >/xx/xx/log.txt

    # dump当前Java程序的堆快照
    $ jmap -dump:format=b,file=/home/mint/heap.hprof 1938
    Dumping heap to /home/mint/heap.hprof ...
    Heap dump file created

### jhat
* 分析Java应用程序堆快照内容

---
    # 分析heap.hprof,放问127.0.0.1:7000
    $ jhat heap.hprof
    Reading from heap.hprof...
    Dump file created Wed Jul 19 23:36:57 CST 2017
    Snapshot read, resolving...
    Resolving 921860 objects...
    Chasing references, expect 184 dots........................................................................................................................................................................................
    Eliminating duplicate references........................................................................................................................................................................................
    Snapshot resolved.
    Started HTTP server on port 7000
    Server is ready.

    # Object Query Language (OQL) query
    select file.path.value.toString() from java.io.File file

### jstack
* 查看Java应用程序的线程堆栈
---

  # 查看线程堆栈信息
  $ jstack -l 1938
  2017-07-19 23:52:04
  Full thread dump Java HotSpot(TM) 64-Bit Server VM (25.131-b11 mixed mode):

  "Attach Listener" #43 daemon prio=9 os_prio=0 tid=0x00007f4ec4001000 nid=0x1c32 waiting on condition [0x0000000000000000]
     java.lang.Thread.State: RUNNABLE

     Locked ownable synchronizers:
  	- None

  "ajp-nio-8009-Acceptor-0" #41 daemon prio=5 os_prio=0 tid=0x00007f4f00582000 nid=0x7e8 runnable [0x00007f4ea90e6000]
     java.lang.Thread.State: RUNNABLE
  	at sun.nio.ch.ServerSocketChannelImpl.accept0(Native Method)

### jcmd
* JDK1.7新增的多功能工具,支持导出堆信息,查看Java进程,导出线程信息,执行GC
---

    # 1938进程所支持的命令
    $ jcmd 1938 help
    1938:
    The following commands are available:
    JFR.stop
    JFR.start
    JFR.dump
    JFR.check
    VM.native_memory
    VM.check_commercial_features
    VM.unlock_commercial_features
    ManagementAgent.stop
    ManagementAgent.start_local
    ManagementAgent.start
    GC.rotate_log
    Thread.print
    GC.class_stats
    GC.class_histogram
    GC.heap_dump
    GC.run_finalization
    GC.run
    VM.uptime
    VM.flags
    VM.system_properties
    VM.command_line
    VM.version
    help

    For more information about a specific command use 'help <command>'.

    # 启动时间
    $ jcmd 1938 VM.uptime
    1938:
    3322.333 s

    # 打印线程信息
    $ jcmd 1938 Thread.print

    # 系统中类的统计信息
    $ jcmd 1938 GC.class_histogram

    # dump堆内存信息
    $ jcmd 1938 GC.heap_dump /home/mint/jdk.dump
    1938:
    Heap dump file created

    # 系统properties信息
    $ jcmd 1938 VM.system_properties
    1938:
    #Thu Jul 20 00:04:01 CST 2017
    java.runtime.name=Java(TM) SE Runtime Environment
    sun.boot.library.path=/opt/jdk/jdk1.8.0_131/jre/lib/amd64
    java.vm.version=25.131-b11
    shared.loader=
    java.vm.vendor=Oracle Corporation
    java.vendor.url=http\://java.oracle.com/

    # 启动参数
    $ jcmd 1938 VM.flags
    1938:
    -XX:CICompilerCount=3 -XX:InitialHeapSize=130023424 -XX:MaxHeapSize=2063597568 -XX:MaxNewSize=687865856 -XX:MinHeapDeltaBytes=524288 -XX:NewSize=42991616 -XX:OldSize=87031808 -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:+UseFastUnorderedTimeStamps -XX:+UseParallelGC

    # 获取性能相关数据

    $ jcmd 1938 PerfCounter.print
    1938:
    java.ci.totalTime=16686437620
    java.cls.loadedClasses=5731
    java.cls.sharedLoadedClasses=0
    java.cls.sharedUnloadedClasses=0
    java.cls.unloadedClasses=22
    java.property.java.class.path="/opt/apache-tomcat-8.0.45/bin/bootstrap.jar:/opt/apache-tomcat-8.0.45/bin/tomcat-juli.jar:/opt/jdk/jdk1.8.0_131/lib/tools.jar"
    java.property.java.endorsed.dirs="/opt/apache-tomcat-8.0.45/endorsed"
    java.property.java.ext.dirs="/opt/jdk/jdk1.8.0_131/jre/lib/ext:/usr/java/packages/lib/ext"

