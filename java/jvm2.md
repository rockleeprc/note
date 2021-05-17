# 类加载过程：
 1、加载
 2、链接（验证、准备、解析）
 3、初始化

 <clinit>() 过程
 1、不需要定义，由编译器自动收集类中的所有变量的赋值动作和static{}中的语句合并而来
 2、如果类有父类，JVM会保证父类的<clinit>()先于子类的<clinit>()执行
 3、JVM保证一个类的<clinit>()多线程下被同步加锁，只会被初始化一次

 类主动使用情况
 1、创建类的实例
 2、访问类或接口的静态变量或对静态变量赋值
 3、调用类的静态方法
 4、反射
 5、初始化一个类的子类
 6、JVM启动时被表明为启动类（包含main()）
 7、Java7 动态语言支持

 类加载器双亲委派：
 1、类加载器收到加载请求，将请求委托给父类的加载器执行
 2、父类加载器存在父类加载器，一次递归委托，知道顶层加载器执行
 3、如果顶层加载可以加载就返回（不会在委托给子加载器），顶层加载器无法加载，子类加载才会尝试加载
 自定义java.lang.String是不会被加载的，最终会被bootstrap classloader加载rt.jar中断的String类
 优点：
 1、避免重复加载
 2、避免核心类库被篡改，保证安全性（不能定义rt.jar中同名的类，不能定义java.lang同名的包）

 在JVM中两个class相等的条件
 1、类的全路径名称必须一致
 2、加载类的ClassLoader必须相同
 在同一个JVM中即使两个class对象来源同一个Class，只要ClassLoader不同，两个class也不想等
 
 # PC Register
 1、对物理PC寄存器的抽象模拟
 2、存储指向下一条指令的地址，由执行引擎读取下一条指令（程序执行的行号指示器）
 3、线程私有，不存在GC，JVM中唯一没有OutOfMemoryError的区域

# 虚拟机栈
背景：Java指令是根据栈设计的，不同平台CPU架构不同，不能设计为基于寄存器，栈是运行时的单位，堆是存储的单位  
优点：1. 跨平台 2. 指令集小
缺点：1. 性能下降 2. 实现同样的通能需要更多的指令
1. 线程私有，一个线程对应一个栈空间，每一个方法对应一个栈帧
2. 保存方法的局部变量、部分 结果，参与方法的调用和返回
3. 不存在GC，出栈操作好比GC 
4. 固定大小虚拟机栈，请求分配的栈空间超过JVM设置的最大容量，StackOverflowError
5. 动态扩展虚拟机栈，在尝试扩展时无法申请到足够的内存或者没有足够内存去创建对应的虚拟机栈，OutOfMemoryError
6. 设置栈空间大小：-Xss 默认1024k
StackFrame
1. 一块内存区域，数据集，保存着方法执行过程中的各种数据信息
2. 每个线程都有自己的栈空间，栈中的数据都是以栈帧的格式存储，线程上每个正在执行的方法对应一个栈帧
3. 方法返回：正常return、出现为捕获的异常 
栈帧内部结构
1、局部变量表
    内部为数字数组，存储方法参数和方法内的局部变量，包括基本数据类型、引用、returnAddress类型
    没有线程安全问题，因为线程私有
    局部变量表大小在编译器确定，运行期不会更改，maximum local variables（局部变量表数量）
    方法嵌套的深度由栈空间大小决定
    基本存储单元是slot，32位占1 slot，64位作占2 slot
    栈帧由构造方法或实例方法创建时，当前对象的引用this会存放在index为0的slot处，其余的参数按照顺序排列
    静态方法内不能使用this是因为局部变量表中没有this引用 
2、操作数栈
    先进后出结构，实现方式：数组、链表
    在方法的执行过程中，根据字节码指令向栈中写入数据或提取数据，主要用于保存计算过程的中间结果，同时作为计算过程中变量临时的存储空间
    栈顶缓存技术：将栈顶元素全部缓存在物理CPU的寄存器中，以此降低对内存的读/写次数，提升执行引擎的执行效率
3、动态链接（指向运行时常量池的方法引用）
    保存当前栈帧指向常量池的方法引用地址，作用是将符号引用转换为直接引用
    静态链接：被调用的目标方法在编译器可知，且运行时保持不变
    动态链接：在编译器无法确定，只能在程序运行期将符号引用转换为直接引用
4、方法返回地址（正常退出或异常退出）
    存储调该用方法的PC寄存器的值，目的是为了继续执行后面的操作 
5、附加信息
    
方法调用指令
    1 invokestatic：调用静态方法，解析阶段确定唯一的方法版本
    2 invokespecial：调用<init>方法、私有方法、父类方法，解析阶段确定唯一版本
    3 invokevirtual：调用所有虚方法（运行时才能确定的方法，但包含调用final方法）
    4 invokeinterface：调用接口
    5 invokesynamic：java8中调用lambda表达式

    对类型检查 1、编译器确认的是静态类型语言 2、运行期确认的是动态类型语言

本地方法接口
    一个native method是java方法，但具体的实现由非java语言实现

本地方法栈
    用于管理本地方法调用的栈空间

# 堆
    一个JVM进程只存在一个堆内存空间，可以处于物理不连续的内存空间
    TLAB（Thread Local Allocation Buffer）
        在eden空间为每个线程分配一个私有的缓冲区，避免多线程操作堆空间地址是时加锁（hotspot采用的是CAS+失败重试的方案）
        这个空间非常小（占eden 1%空间），不是所有对象都能在tlab中，但是在分配空间时tlab是首选的空间
        -XX:UseTLAB 默认开启
        整个堆空间只有tlab不是共享的，其它空间都是共享的
    方法调用结束后，堆空间的内存不会被马上被回收，只有在GC时才会被回收
    创建对象的指令new、newarray
    java7及以前：新生代（Eden、Survivor1、Survivor2）+ 老年代（Old/Tenure）+ 永久代（Perm）
    java8及以后：新生代（Eden、Survivor1、Survivor2）+ 老年代（Old/Tenure）+ 元空间（Meta）
    默认最小堆内存：物理内存/64 
    默认最大堆内存：物理内存/4
    堆空间参数设置：
        设置堆空间大小（建议ms、mx设置成一样的值，避免GC后堆空间大小动态调整）
            -Xms  -Xmx 
        设置新生代、老年代比例
            -XX:NewRatio=2（默认） 新生代占1，老年代占2，新生代占老年代的1/3
            -XX:NewRatio=4 新生代占1，老年代占4，新生代占老年代的1/5 （只有明确知道对象存活时间比较长时，才会修改这个比例）
        设置新生代eden、survivor比例
            -XX:SurvivorRation=8(默认) 8:1:1设置Eden大小，自适应内存策略会动态调整这个值，必须显示使用这个设置才会生效
            eden空间比较小会频繁触发Minor GC，survivor空间比较小会使对象晋升老年代几率增加，间接导致Full GC时间增常
        新生代年龄最大值

        设置新生代空间大小
            -Xmn （一般都设置比例大小，很少直接设置新生代大小）
        查看参数设置
            jinfo -flag 参数名称 pid // 查看运行时jvm进程设置的参数值
            jinfo -flags 16589 // 查看jvm启动参数配置
            -XX:+PrintFlagsInitial // 查看所有参数的默认值
            -XX:+PrintFlagsFinal // 查看所有参数的最终值
    查看堆空间大小：
        jstat -gc pid 
        -XX:PrintGCDetails
    对象分配过程：

    Young GC/Minor GC：
        只针对新生代，eden满了触发（survivor满了不会触发），eden存活的对象会被晋升到survivor，survivor中的对象会记录age，age计数器默认为15，大于15的对象会被晋升到old
        会引发STW
    Major GC/Old GC：
        只针对老年代，只有CMS GC才会单独收集老年代
        执行Major GC时通常会伴随一次Minor GC，但比Minor GC慢10倍，Major GC后内存还不足就OOM
        会引发STW，STW时间更长
    Mixed GC：
        针对新生代和部分老年代，目前只有G1 GC会有这种行为
    Full GC：
        针对整个堆和方法区/元数据
        触发场景：
            1. 调用System.gc()时，建议JVM执行Full GC，但不是必须执行
            2. 老年代空间不足时
            3. 方法区空间不足时
            4. Minor GC后进入老年代对象的大小超过老年代可以用空间
            5. eden对象进行s0、s1复制对象时，对象大小超过survivor空间大小，晋升老年代时，老年代空间又不够 

    堆空间分代收集思想：
        对象的生命周期不同，将不同生命周期的对象放到不同的空间，优化GC效率

    逃逸分析（C2编译器，server模式才有）：
        目的是避免在堆上创建对象，关闭逃逸分析（默认开启）：-XX:-DoEscapeAnalysis
        1、栈上分配：
            JIT通过逃逸分析发现一个对象的作用域只在方法内部，就有可能被优化为栈上分配（标量替换），无需在堆上分配内存，避免GC
        2、锁消除（同步省略）：
            JIT编译器通过逃逸分析检查同步代码块是否只能被一个线程访问，没有被发布到其它线程，这时编译器会取消代码的同步提高性能，字节码层面还是会保留monitor
            默认开启
        3、标量替换（分离对象）
            有的对象可以不需要连续的内存结构存储，对象的一部分或全部可以不存在于内存中，而是存在于CPU寄存器中（java就是栈中，将对象打散分配到栈上）
            默认开启
            标量即不可被进一步分解的量，而JAVA的基本数据类型就是标量（如：int，long等基本数据类型以及reference类型等）
            聚合量是可以被进一步分解的量，而这种量称之为，而在JAVA中对象就是可以被进一步分解的聚合量。
            通过逃逸分析确定该对象不会被外部访问，并且对象可以被进一步分解时，JVM不会创建该对象，而会将该对象成员变量分解若干个被这个方法使用的成员变量所代替，这些代替的成员变量在栈帧或寄存器上分配空间

# 方法区 
    在JVM启动时创建，保存系统中的类信息，溢出会导致oom（加载三方jar包、tomcat部署工程过多、大量的反射操作）
    在JVM规范里方法区和永久代并不等价，hotspot在实现层面上是等价的，元空间与永久代最大的差别是元空间使用本地内存
    
    设置元空间大小：
        --XX:PermSize 默认20.75M
        --XX:MaxPermSize 默认82M（64位）
        --XX:MetaspaceSize 默认21M
        --XX:MaxMetaspaceSize 默认-1，本地内存的最大值
    
    高水位线：

    类型信息:
        类、接口、枚举、注解、修饰符、方法信息、域信息、异常表 
    常量:
        .class中的常量池：
            可以看作一张表，JVM根据指令查找常量池表找到要执行的类名、方法名、参数类型、字面量等类型
            .class文件中的一部分，在编译的.class中就会存在，用于存放编译器期生成的各种字面量与符号引用，这部分内容将在类加载后存放到方法区的运行时常量池中
        运行时常量池：
            方法区的一部分，在加载类和接口到JVM后，就会创建对应的运行时常量池
            比起常量池具有动态性，比如String.intern()
    静态变量:
        在prepare阶段会初始化默认值，在初始化阶段在会赋值，编译的class中不会有值
        static final的变量在编译器就会被分配，并在.class文件中可以看到
    即时编译器编译后的代码缓存:

    jdk8及之后类信息、字段、方法、常量保存在本地内存的元空间，字符串常量池、静态变量仍在堆内存

# 对象实例话与访问
    对象实例化
        创建对象的方式
            1. new
            2. Class.newInstance()：只能调用无参构造器，权限必须是public，JDK9以后废弃
            3. Constructor.newInstance(xxx)：JDK9以后反射建议使用此种方式
            4. clone()
            5. 反序列化
            6. 三方库操作字节码

        对象实例化过程
            1. 加载类元数据信息
            2. 为对象分配内存
            3. 处理并发问题
            4. 属性的默认初始化（零值初始化）
            5. 设置对象头信息
            6. 属性的显示初始化、代码块初始化、构造器初始化

    对象内存布局
        对象头（Mark Word）：
            运行时元数据：哈希值、GC分代年龄（age）、锁状态标志、线程持有的锁、偏向线程ID、偏向时间戳
            类型指针：指向元数据InstanceClass，确定对象所属的类型
            如果是数组还需要记录数组长度
        实例数据：对象真正存储的有效信息，字段类型
        对齐填充：占位符

    对象访问
        1. 句柄访问
        2. 直接指针（hotspot采用的方式）

    直接内存
        不是JVM运行时数据区的一部分，也不是JVM规范中定义的内存空间
        直接在JVM堆外向操作系统申请的空间，通过堆中的DirectByteBuffer对象操作native内存
        读写频繁的场景适合使用直接内存
        分配成本较高，不受JVM内存回收限制
        配置最大直接内存：
            -XX:MaxDirectMemorySize=xx 
            默认直接内存大小于-Xmx一致，也可能OOM，但不会受限于-Xmx影响，受限于系统内存空间
    
# 执行引擎
    作用就是将字节码指令解释/编译为对应平台的本地机器指令
    javac编译.java文件属于前端编译，执行引擎编译.class文件属于后端编译    
    解释器：根据JVM规范对字节码逐行解释执行，将字节码翻译为本地机器指令执行
        java -Xint 
    JIT编译器：JVM将源代码直接编译为本地机器指令
        java -Xcom
        java -Xmixed 默认模式
        hotspot中默认有两个JIT编译器，分别为client compiler（c1编译器）、server compiler（c2编译器），c2比c1编译器更深度优化，编译时间较长，优化策略更激进
        java -client
        Java -server  

    高级语言 --> 汇编语言 --> 机器指令 --> CPU

    热点代码探测：
        方法调用计数器：统计方法调用次数
            client模式默认1500 server模式默认10000，超过阀值会触发JIT编译器
            --XX:CompileThreshold 设置阈值
        回边计数器：统计循环体执行次数

# String
    Java8及以前使用char[]，Java9及以后使用byte[]
    StringTable中不会存储相同的内容的字符串，Java8中存在于堆空间，Java8前存在于永久代 
    StringTable设置：
        -XX:StringTableSize
    
    字符串拼接操作
        1. 常量于常量的拼接结果在常量池，在编译器优化（final修饰的String拼接，直接从常量池里获取）
        2. 常量池中不会存在相同的字符串常量
        3. 只要拼接过程中出现变量（非final），使用StringBuilder调用append方法进行拼接，然后toString()返回
        4. 字符串拼接结果调用inter()则将结果存入常量池

        String s1 = "a"; // 字面量直接进入常量池
        String s2 = "b";
        String s3 = s1 + s2; // 拼接操作，不会进入常量池
        String s4 = "ab";
        此时s3、s4不相等，如果s1、s2使用final修饰将相等

        String s = new String("ab");// 字符串常量池里有ab
        String s = new String("a")+new String("b");
            5个对象：// 字符串常量池中没有ab，通过StringBuilder.append()实现
                1. s
                2. new String()
                3. "a" 常量池
                4. new String()
                5. "b" 常量池
                6. new StringBuilder()

        面试题：（理解过程大于结果）
        ```java
            String s1 = new String("a");
            s1.intern();
            String s2 = "a";
            System.out.println(s1 == s2);

            String s3 = new String("2") + new String("2");
            s3.intern();
            String s4 = "22";
            System.out.println(s3 == s4);
        ``` 
    
        打印常量池GC信息：
            -XX:+PrintStringTableStatistics

# 垃圾回收
    垃圾：运行程序中没有任何指针指向的对象
    垃圾回收标记阶段的目的是判断对象是否存活，被标记为死亡的对象才会被GC收集，判断对象存活的算法：引用计数器、可达性分析

## 引用计数算法（标记阶段）
    描述：
        对每个对象保存一个整型的引用计数器，用于对象被引用的次数
    优点：
        实现简单，判断效率高，回收没有延迟
    缺点：
        计数器增加对象的存储空间
        针对计数器操作的加减开销
        无法处理对象间循环引用的问题
        python使用这种计数器，解决循环依赖的办法：手动解除、若引用
## 可达性分析算法（根搜索算法）（标记阶段）
    描述：
        GC Roots：一组必须活跃的对象
            JVM虚拟机栈中的变量
            方法区中静态属性
            方法区中的常量对象
            synchronized持有的对象
            基本数据类型对应的Class对象
            常驻的异常
            系统类加载器
        以GC Roots为起点，按照从上到下的方式搜索被GC Roots对象所链接的目标对象是否可达
        算法计工作时必须在一个能保证一致性快照中进行，所以才会导致STW
    优点：

    缺点：

## 标记-清除算法（清除阶段）
    标记：从根结点遍历，标记所有被GC Roots引用的多想，在对象头中记录对象可达
    清除：从堆内存从头到尾进行线性遍历，发现某个对象头中没有标记为可达，则将其回收（不是真的删除对象，而是将对象的地址放入到空闲列表中）
    缺点：
        效率不高，每个阶段都需要遍历堆内存；GC时会STW；清理后产生内存碎片（需要单独维护空闲列表）

## 复制算法（清除阶段）
    将内存空间分为两块，每次只使用一块，在GC时将存活的对象复制到另外一块内存空间，将前一块内存空间中的数据清除，避免内存碎片
    新生代中的Survivor区使用这种算法
    优点：
        没有标记、清除过程，效率高，实现简单；复制过程没有碎片
    缺点：
        内存空间利用率1/2
    不太适合大量对象存活的情况，复制大量存活对象没有性价比（新生代）

## 标记-压缩算法（清除阶段）
    标记：
    压缩：将所有存活的对象压缩的内存的一端（移动对象，没有碎片）
    优点：
        与标记-清除算法比内存无碎片；与复制算法比使用完整内存空间
    缺点：
        效率上低于复制算法；对象压缩过程需要修改引用地址；复制过程需要STW

## 分代收集算法
    没有一种算法是完美的，不同生命周期的对象采用不同的垃圾回收策略，以便GC的效率更高效
    新生代：复制算法
        特点：区域相对老年代较小，对象生命周期短、存活率低、回收频发
    老年代：标记清除、标记压缩
        特点：区域较大，对象生命周期长、存活率高，回收相比新生代不频繁
        mark：开销与存活对象多少成正比
        sweep：开销与所管理的内存区域大小成正比
        compact：开销与存活对象多少成正比

## finalization机制
    对象三种状态
        1. 可触及：从GC Roots可以到达的对象
        2. 可复活：对象所有引用都被释放，GC Roots不可达，但对象可能在finalize()中复活
        3. 不可触及：对象的finalize()被调用，并且没有复活，finalize()只会被JVM调用一次

# 垃圾回收

## System.gc()
    不会保证对垃圾收集器的调用，开发中使用的场景较少
    System.runFinalization(); // 强制调用失去引用对象的finalize()

## OOM
    javadoc：没有空闲的内存，并且GC也无法提供更多内存
    方法区OOM：
        JVM虚拟机设置参数不合理
        程序中创建了大量的对象，并且长时间不能被GC回收

## Memory Leak
    对象不会再被程序使用到，但GC又回不能回收（严格的概念）
    由于编码习惯不好导致对象的生命周期变得很长（宽泛意义上的泄漏）
        单例对象引用外部对象
        资源没有及时close()

## Stop The World
    GC时应用程序所有线程都会被暂停，可达性分析算法必须确保数据的一致性
    所有的GC都会STW，无法避免这种情况，只是将这频率降低，时间减少

## 并行与并发
    并发：同一时间段，单核CPU，资源抢占
        用户线程和垃圾收集线程同时执行，CMS、G1
    并行：同一时间点，多核CPU
        多条垃圾收集线程并行工作，用户线程处于等待状态，ParNew、Parallel Scavenge、Parallel Old

## Safe Point
    程序执行并非可以随时GC，只有在特定位置时才可以，这个位置就是Safe Point
    Safe Point太少可能导致GC等待时间太长，太多可能导致GC过于频繁， 方法调用、循环跳转、异常跳转
    Safe Region：
        线程sleep、block状态时无法响应中断，不能到达Safe Point
        Safe Region指在一段代码片段中，对象的引用关系不会发生变化，这个区域的任何位置都GC都是安全的

## 强 Strong
    无论在任何情况下GC都不会回收
    强引用的状态都是可达的
    强引用对象是内存泄漏的主要对象
## 软 Soft
    JVM将要发生OOM前，会把这些对象列入回收范围进行二次回收（第一次是不可GC Roots不可达对象），如果GC后还没有足后内存就OOM
    只有内存不够时才会被GC
    声明时一定要将强引用对象设置为null，只保留软饮用对象
## 弱 Weak
    只能生存到下一次GC前，无论内存空间是否足够都会被回收
    
## 虚 Phantom
    用于追踪对象垃圾回收的过程

# 垃圾收集器

## Serial
    串行，新生代，复制算法，client模式下默认的收集器，STW机制

## Serial Old 
    串行，老年代，标记压缩算法，client模式下默认的收集器，STW机制
    java8 作为cms的后背方案，与Parallel Scavenge配置使用
    多核CPU单线程效率不高，单线程高效,java web不会采用这种GC（STW太明显）
    配置：
        -XX:+UseSerialGC 将新生代和老年代都指定为Serial
## ParNew
    并行，新生代，Serial的多线程版本，复制算法，STW机制（比Serial时间短，毕竟多线程）
    java8 老年代可以使用CMS、Serial Old
    单核CPU没有Serial好
    配置：
        -XX:+UseParNewGC 新生代使用ParNewGC
        -XX:ParallelGCThreads 设置并行线程数量，默认和CPU核数一致    

## Paralle Scavenge
    并行，新生代，复制算法，STW机制，吞吐量优先（不需要太多与用户交互的场景，批处理、定时任务）
    java8 默认的收集器 

## Paralle Old
    并行，老年代，标记压缩，STW机制，吞吐量优先（不需要太多与用户交互的场景）
    java8 默认的收集器
    可以配置暂停时间，比例，建议使用自适应策略

## CMS
    并发（用户线程、GC线程同时工作），老年代，标记清除算法（无法解决内存碎片），STW机制，低延时（区别吞吐量，适合与用户交互程序）
    新生代使用ParNew、Serial
    GC过程：
        初始标记：STW
        并发标记：用户线程和标记线程共同执行（GC线程数=（Thread+3/4））
        重新标记：STW
        并发清理：用户线程和标记线程共同执行
    缺点：
        产生内存碎片
        并发标记时占用系统线程
        触发处理浮动垃圾（在并发标记时出现新的垃圾无法被标记，只能下一次GC时回收）

    配置：
        -XX:+UseConcMarkSweepGC // 老年代使用CMS 新生代使用ParNew
                                // 默认Java8 92% 触发cms GC

## G1
    把堆内存分割为多个Region，物理上可以不连续，使用不同Region表示Eden、Survivor、Old，跟踪Region里面的垃圾价值，保存在优先列表中，优先回收价值高的Region
    针对多核CPU和大容量内存
    优点：
        并行与并发、分代收集、空间整合、可预测的停留时间模型
    适用于大容量堆内存6GB或以上、低延迟场景
    配置：
        -XX:+UseG1GC // 使用G1

## GC日志
Serial [DefNew]
ParNew [ParNew]
Parallel Scaveng [PSYoungGen]
Parallel Old [ParOldGen]

# 字节码指令
    成员变量赋值过程
        默认初始化
        显示初始化/代码块初始化
        构造器初始化
        对象调用设值
    
    字节码=操作数+操作吗（有些语言没有操作数概念）

    每个类一个字节码 jclasslib、javap

## class文件结构
     魔术
        4个字节无符号整数
     Class文件版本
        major version、minor version
        java6 50 0
        java8 52 0
     常量池
        常量池计数器从1开始，具体内容范围 1 ～ constant_pool_count-1
        存放字面量和符号引用，内容会被加载到运行常量池中
        字面量：文本字符串、声明final的常量值
        符号引用：类和接口的全限定名、字段名称和描述符、方法名称和描述符
     访问标志
     类索引、父类索引、接口索引集合
     字段表示集合
     方法表示集合
     属性表示集合

## 字节码

    执行模型
        do{
            自动计算PC寄存器的值+1
            根据PC寄存器的位置，从字节流中取出操作码
            if(字节码存在操作数)
                从字节流中取出操作数
            执行操作码定义的操作
        }while(字节码长度>0)

    助记符中的数据类型
        i   int、boolean
        l   long
        s   shor
        b   byte
        c   char
        f   float
        d   double
        a   引用类型

    加载与存储指令：将数据从栈帧的局部变量表和操作数栈之间来回传递
    局部变量入栈指令
        xload、xload_<n>：将局部变量表的数据压入操作数栈（n=索引位置）
    常量入栈指令
        xconst_<n>：常量数据压入操作数栈（n=实际值）（iconst_x -1,5）
        aconst_null：引用类型位null
        bipush：8位整数参数（-128,127）
        sipush：16位整数参数 （-32768,32767）
        ldc：以上指令都不行，8位参数（指向常量池中的int、float、String的索引值）
        ldc_w：两个8位参数（索引值长度）
        ldc2_w：入栈long、double
    出栈入局部变量表指令
        xstore、xstore_n：出栈操作，将栈顶元素弹出后存入局部变量表，用于给局部变量表赋值（n=索引位置）
    算数指令

    TODO 249    



# 类的生命周期

    加载
        将Java类的字节码文件加载到内存，并构建出Java类的模版对象（方法区），生成Class的实例（堆）
        加载源：文件系统、jar、zip、http、运行时生成Class
        数组不是由类加载器创建的，是由JVM在运行时根据需要直接创建的
    验证
        保证加载字节码合法
        格式检查：魔术检查、版本检查、长度检查
        语义检查：是否继承final、是否有父类、抽象方法是否有实现
        字节码验证：
        符号引用验证：
    准备
        为静态变量分配内存（非final），并初始化默认值
        JVM不支持boolean类型，使用的是int类型，int默认值0，非0都为真，0是假，boolean默认值为false
        没有具体代码的执行，执行代码在初始化阶段
        static final 基本数据类型、static final String 确定的值
    解析
        将类、接口、字段、方法的符号引用转换为直接引用
    初始化
        为静态变量赋值正确的初始值
        执行<clinit>()类初始化方法，由Java编译自动生成，JVM执行调用（static静态块，静态变量赋值），优先调用父类<clinit>()
        static final 类型=不确定的值
        <clinit>()带锁线程安全
        主动初始化时才会调用clinit，被动初始化时不会
        Class.forName() 主动    ClassLoader.loadClass() 被动
    使用

    卸载

TODO 283



# 深入理解JVM虚拟机 第三版

对象创建
    指针碰撞
    空闲列表
    TLAB
    CAS

对象定位方式
    句柄池
    直接指针

方法区、永久代、常量池
堆
虚拟机栈、本地方法栈
程序计数器

# JVM调优

## jps
    -m 显示参数
    -v 虚拟机启动参数

## jstat
    显示类装载、内存、垃圾收集、JIT编译
    没有GUI时，定位问题的首选工具
    jstat -<option> [-t] [-h<lines>] <vmid> [<interval> [<count>]]

    -class // 与ClassLoader相关的信息（类装载、卸载、总空间、耗时）
    -compiler // 显示编译信息
    -gc  // 堆相关的gc信息
    -gcutil // 堆相关的gc比信息
    -gccause // 显示gc的原因

    -t  // 打印信息显示时间戳列
    -h  // 多少行显示表头信息 -h3 没个3行显示表头

## jinfo
    查看JVM配置参数，调整配置参数
    -syspros // 查看System.getProperties()取得的参数
    -flags // 经过赋值的参数(jVM启动参数)
    -flag 具体参数 pid //  查看具体参数值
    -flag -/+ 具体参数 pid // 修改参数值，只能修改manageable的参数
    
## jmap
    导出内存快照文件、查看内存使用情况
    在安全点才会dump，如果有线程无法到达安全点，dump将一直等待
    -dump
        # dump 内存快照
        -dump:format=b,file=<filename.hprof> pid
        # dump 存活的对象
        -dump:live,format=b,file=<filename.hprof> pid
        # oom时自动dump
        -XX:+HeapDumpOnOutOfMemoryError
        -XX:HeapDumpPath=<filename.hprof>
    -heap # 堆空间占用情况
        堆空间大小、使用情况
    -histo # 内存对象信息
    -permstat # 查看系统的ClassLoader信息
    -finalizerinfo # 查看finalizer队列中的对象

## jhat
    分析jamp -dump文件
    建议实用工具分析dump文件，java9中该命令已经被移除

## jstack
    线程快照信息
        Dealloc 死锁
        Waiting on condition 等待资源
        Waiting on monitor entry 等待获取监视器
        Blocked 阻塞
        Runnable 执行
        Suspended 暂停
        Wating 等待
        Time_Wating 
    -m 显示调用本地方法的堆栈信息，C/C++程序的堆栈
    -l 显示关于锁的附加信息
    -F 进程不响应时，强制输出线程堆栈

    Thread.getAllStackTraces() // 通过代码显示线程信息

## jcmd
    实现除jstack外所有的功能
    建议用jcmd替换jmap    
    
    -l 列出所有JVM进程 替换jps
    jcmd pid help 针对pid所支持的命令

## 扩展参数
    # 查看JVM参数的最终值 被:=标记的为修改后的值   
    java -XX:+PrintFlagsFinal -version | grep manageable
    # 查看所有JVM参数启动的初始值
    java -XX:+PrintFlagsInitial 
    # 查看被用户或JVM设置的参数及值
    java -XX:+PrintCommandLineFlags

# JUI工具
    JDK自带工具
        jconsole
        visualVM
        JMC
    三方工具
        MAT(eclipse 插件)
            浅堆：不包含数据
            深堆：保留集（唯一引用）+浅堆
        JProfiler 付费
        Arthas 开源
        Btrace



1字=4Bytes
001 normal
101 biased
00 轻量级
10 重量级
11 mark word for gc

轻量级锁
    多个线程访问同一把锁，当没有出现竞争时，可以使用轻量级锁进行优化
    加锁流程：
        1、在线程栈帧中创建lock record（lock record地址+00、锁对象指针）
        2、使用cas将lock record中的lock record地址与锁对象中的mark word进行交换，表示加锁（修改锁记录表示00），锁对象mark word的锁状态必须是无锁时cas才会成功
        3、失败情况
            1、锁对象已经被占用，表死存在竞争，进入锁膨胀
            2、自己持有锁并且重入了，在栈帧中会再添加一个lock record对象，mark word设置为null，锁对象指针指向锁，lock record表示锁重入次数
            3、t1持有obj轻量级锁，t2获取锁失败：
                1、t2进入锁膨胀，为obj申请monitor锁，将obj的mark workd指向monitor地址，将锁状态设置为10，t2进入EntryList等待                
        4、轻量级解锁：
            1、使用cas将lock record里的mark word设置回锁对象的mark word中，cas成功将解锁成功
            2、cas失败说明锁升级为重量级锁，进入重量级解锁流程
        

偏向锁
    解决问题：
        锁重入时，轻量级锁执行cas操作，用lock record替换mark word带来的消耗
    实现：
        1、线程第一次加锁时使用cas将线程id记录到mark word中，只要以后不发生竞争，这个锁永远属于该线程所有
        2、偏向锁延迟启动，JVM启动时不会马上生效
        3、偏向锁启动后，不会主动释放，通过其它竞争线程将锁设置为轻量级锁（线程顺序执行）、重量级锁（线程竞争）
    撤销：
        1、当调用锁对象的hashCode()后，将会撤销偏向锁，原因是hashcode存在于mark word中（占32bit），当mark word中保存线程id（占54bit）后就没有更多的空间保存其它属性，轻量级锁、重量级锁hashcode存在于lock record中
        2、t1持有偏向锁，释放锁后偏向状态不会消失，t2申请加锁，此时锁的mark word偏向t1，t2将锁偏向锁撤销，升级为轻量级锁也可能重偏向t2，此时没有竞争，t1、t2顺序执行
        3、wait()、notify()也会导致偏向锁撤销，只能持有重量级锁才能调用
    批量重偏向：
        锁对象被多个线程访问，单没有出现竞争，锁偏向t1后仍有机会偏向t2，重偏向会修改mark word中的线程id
        锁偏向t1 -> 撤销 -> 
                            t2轻量级锁 -> 
                                        释放 -> t2轻量级锁 -> 
                                                            释放 -> t2轻量级锁 -> 
                                                                                 锁偏向t2
    批量撤销：
        撤销偏向锁阀值超过40次后，JVM认为可能不该偏向某个线程，锁对象就变为不可偏向




