# jvm连载：内存结构
## tips
* jvm不像代码那样对程序员很直观，基础概念和理论又特别多
* 本着理解记忆，了解每块概念，多看多计
* 着重理解jvm运行时数据区，每个区域的作用，是否线程私有，有什么异常

## 程序计数器
* jvm在执行指令时需要知道要执行哪条指令，计数器存储的就是下一条要执行的指令地址
* 具体的工作过程：解释器在解析jvm指令时，根据计数器的地址依次执行指令，将指令翻译为机器码，计数器就是jvm内存模型对硬件寄存器的抽象实现
* 多线程执行jvm指令时，每个线程的计数器是独立的，互不影响
* 该区域没有任何OutOfMemoryerror

## 虚拟机栈
* 描述方法调用的执行过程，每个方法在执行时，会将方法参数、局部变量、操作数栈、方法出口等信息封装为栈帧（Stack Frame）
* 方法的调用对应着栈帧的入栈和出栈
* 栈帧的核心是局部变量表，存在编译期可知的Java基本类型数据、对象引用、returnAddress类型，这些数据存储的单位成为slot，64位长度的long、double占两个slot，其余的占一个slot
* 局部变量表空间在编译期完成分配，运行期不会改变
* 超过调用栈最大深度抛出StackOverFlowError，无法申请到内存抛出OutOfMemoryerror

## 本地方法栈
* 作用与虚拟机栈一致，hotspot将本地方法栈和虚拟机栈合二为一，没有单独实现本地方法栈
* 超过调用栈最大深度抛出StackOverFlowError，无法申请到内存抛出OutOfMemoryerror

## 堆
* 该区域唯一的目的就是存放对象，在GC分代收集思想下，堆又被各个垃圾收集器划分为新生代、老年代、永久代...等多个区域
* 线程共享，但是为了提高对象分配效率，TLAB可以划分出线程私有的堆空间
* 无法申请到内存抛出OutOfMemoryerror

## 方法区
* 存储类加载信息、常量、静态变量等
* 在jdk1.8前，方法区的实现方式是用永久代，当时两者并不等价，只是由于在GC分代思想下，将方法区设置在永久代中方便垃圾收集器管理
* 从jdk1.8开始，完全废弃永久代概念，将类的加载信息移到元空间，将字符串常量池、静态变量移到堆空间
* 无法申请到内存抛出OutOfMemoryerror

### 运行时常量池
* 方法区的一部分，Class文件中有一个区域叫常量池，用于存放编译期生成的字面量和符号引用，这部分内容在类加载后存储到运行时常量池
* 运行时常量池具有动态扩展性，比如调用String.intern()将字符串加入到常量池，但池Class中的常量池编译期就可获得，没有扩展性
* 无法申请到内存抛出OutOfMemoryerror

## 直接内存
* 不是java虚拟机规范中定义的区域，使用本机内存，无法申请到内存抛出OutOfMemoryerror

## new指令执行过程
1. 检查指令参数在常量池中是否有一个符号引用
2. 检查符号引用的类是否已被加载、链接、初始化
3. 如果没有加载，必须先执行加载过程
4. 为对象分配内存，对象所需的内存大小在类被加载完成后便可确定
5. 将对象中的字段都初始化为零值
6. 设置对象头信息：哪类的实例、对象的哈希吗、GC分代年龄、锁信息
7. 调用`Class<init>()`执行构造方法初始化

## 对象内存分配方式
### 指针碰撞
* 堆内存空间绝对规整，没有内存碎片
* 被使用过的空间放在左边，空闲的内存放在右边，中间使用指针作为分界点，分配内存时仅仅是把指针向右移动一段对象大小空间
* 带压缩整理的垃圾收集器使用这种方式
### 空闲列表
* 使用内存和空闲内存交错在一起
* jvm维护一个列表，记录哪块内存可用，在分配时找到一块足够大小的空间分给对象
* 没有压缩真理的垃圾收集器使用这种方式
### TLAB
* 在jvm中对象创建是频繁的操作，即使只修改指针的位置都有可能造成线程不安全
* 为每个线程预先在堆内存空间中分配一小块区域，分配空间时优先使用这块空间
* TLAB空间用完后，分配内存时采用cas+失败重试保证原子性操作

## 对象内存布局
* 对象头
    * mark word：哈希码、GC分代年龄、锁状态标志
    * 类型指针：类型元数据的指针，确定该对象是哪个类的实例
    * 长度：如果是数组类型还必须记录数组的长度
* 实例数据
    * 类型字段信息
* 对齐填充
    * 占位符，对象的大小必须是8的倍数


# jvm连载：垃圾回收
## GC Root
## 引用
### 强引用
* 由GC Root关联的对象都是前引用对象，无法被回收
### 软引用
* 在GC后内存空间仍然不足时，会再次触发GC回收软引用的对象
* 可以配合引用队列释放软饮用自身对象
### 弱饮用
* 在GC时无论内存是否够用，软饮用对象都会被回收
* 可以配合引用队列释放软饮用自身对象
### 虚引用
* 必须配合引用队列使用
* 主要配置ByteBuffer使用 // TODO
### 终结器引用
* 不需要编码，必须关联引用队列使用
* 在第一次GC时，对象的终结器引用加入引用队列，由优先级很低的Finalizer线程通过终结器引用调用finallize()，第二次GC时才能回收该对象
### 引用队列

## 垃圾回收算法



## 
```java
String s1 = "a";
String s2 = "b";
String s3 = "ab";
String s4 = s1 + s2; // 变量在运行时可能被修改
System.out.println(s3==s4);
```
## 
```java
String s1 = "a";
String s2 = "b";
String s3 = "ab";
String s4 = "a" + "b"; // 编译期间结果确定（编译器优化）
System.out.println(s3==s4);
```  
## 
```java
final String s1 = "a";
final String s2 = "b";
String s3 = "ab";
String s4 = s1 + s2; // s1、s2是常量时，编译期结果可以确定
System.out.println(s3==s4); 
```    
## 
```java
String s1 = new String("a")+new String("b");
s1.intern();
String s2="ab";
System.out.println(s1==s2);
System.out.println(s1=="ab");
```
在程序一开始声明String s="ab"，intern()返回的是字符串常量池的对象
1.8 intern()
    string table存在，不放入
    string table不存在，放入string table，返回string table中的对象
1.6 inter()
    string table不存在，复制对象放入string table，返回string table中的对象 
    String s = "a";
    s==s.intern(); // false

## 延迟初始化
```java
System.out.println("1");
System.out.println("2");
System.out.println("3");
System.out.println("4");
System.out.println("5");
System.out.println("6");
System.out.println("7");
System.out.println("8");
System.out.println("9");
System.out.println("10");

System.out.println("1");
System.out.println("2");
System.out.println("3");
System.out.println("4");
System.out.println("5");
System.out.println("6");
System.out.println("7");
System.out.println("8");
System.out.println("9");
System.out.println("10");
```