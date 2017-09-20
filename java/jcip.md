
# 第一章 简介
## 并发简史
* 进程：独立的资源，包括内存、文件句柄、安全证书
* 线程：同一个进程内共享资源，包括内存，文件句柄，但每个线程都有各自的程序计数器、栈、局部变量
* 由于同一个进程中的所有线程都将共享进程的内存，因此这些线程都能访问相同的变量并在同一个堆上分配对象，这就需要实现一种比在进程间共享数据粒度更细的数据共享机制

## 线程的优势
* 发挥多核处理器能力
* 建模的简单性
* 异步事件的简化过程
* 响应更灵敏的用户界面

### 安全性
* 读-操作-写（非原子性）
* 多线程中的操作执行顺序是不可预测的
* 多个线程同时访问和修改相同变量时，在串行模型中引入非串行因素

### 活跃性
* 安全性：永远不发生糟糕的事
* 活跃性：某件正确的事情最终会发生，但不够好
	* 串行程序：死循环后的代码不会被执行
	* 并发程序：死锁、饥饿、活锁

### 性能问题
* 调度器临时挂起一个活跃的线程并执行另一线程时，线程间上下文切换，当并发量大时，这种频繁切换上下文会带来极大的开销


# 第二章 线程安全性
## 什么是线程安全性
* 当多线程访问某个类时，这个类始终都能表现出正确的行为，这个类就是线程安全的
* 线程安全类中封装了必要的同步机制，客户端无须进一步采取同步措施
* 无状态对象一定是线程安全的（没有局部变量的Servlet）
* 多个线程之间的操作无乱采用何种执行时序或交替方式，都要保证不变性条件不被破坏

## 原子性
* 竞态条件：由于不恰当的执行时序而出现不正确的结果，当某个计算的正确性取决于多个线程的交替执行时序时，就发生竞态条件
* 复合操作：先检查后执行、读取-修改-写入都是复合操作，这些操作必须以原子方式执行以确保线程安全性
* 竞态条件类型：
	* 先检查后执行（Check-Then-Act），基于一种可能失效的观察结果来做出判断或这执行某个计算（延迟初始化）
	* 读取修改写入（Read-Edit-Write），基于对象之前的状态定义对对象的转换（计数器递增）
	* 不存在则添加（Put-If-Absent），判断某个状态，根据状态确定是否要插入数据
* 在无状态的类中添加一个状态变量时，如果该状态变量是线程安全的类，这个类仍然是线程安全的

## 加锁机制
* 当不变性条件中涉及多个变量时，各个变量之间并不是彼此独立的，而是某个变量的值会对其它变量的值产生约束，要保持状态的一致性，需要在单个原子操作中更新所有相关的状态变量

### 内置锁
* synchronized
	* 锁对象的引用
	* 锁保护的代码块
* 内置锁互斥：当线程A获取线程B持有的锁时，线程A必须等待或阻塞，直到线程B释放这个锁，如果线程B不释放，线程A将永远等待下去
* 任何一个执行同步代码块的线程，都不可能看到由其它线程正在执行由同一个锁保护的同步代码块

### 重入
* 内置锁是可重入的，如果某个线程试图获得一个已经由自己持有的锁，那么这个请求会成功

## 用锁来保护状态
* 锁能使用其保护的代码以串行形式来访问，对共享状态的复合操作，都必须是原子性操作以避免产生竞态条件
* 对于可能被多个线程同时访问的可变状态变量，访问它时都需要持有同一个锁（读取时加锁）
* 每个共享的可变变量都应该只由一个锁来保护，从而使维护人员知道是哪个锁
* 获取与对象关联的锁时，并不能阻止其它线程访问该对象，某个线程获得对象的锁之后，只能阻止其它线程获得同一个锁

## 活跃性与性能
* 缩小同步代码块的作用范围，不要将原子操作拆分到多个同步代码块中，尽量将不影响共享状态且执行时间较长的操作从同步代码块中分离出去
* 当执行时间较长的计算或者可能无法快速完成的操作时，一定不要持有锁


# 第三章 对象共享
## 可见性
* 同步机制不仅保证操作的原子性，还确保多个线程之间对内存写入操作的可见性
* 在没有同步的情况下，编译器、处理器以及运行时等都可能对操作的执行顺序进行一些意想不到的调整

### 失效数据
* 对于共享数据的get/set时，一定要是原子性
* 线程间操作共享数据时，一定要保证可见性

### 非原子的64位操作
* 最低安全性：在没有同步下读取变量时，可能得到一个失效的值，但至少这个值是由之前某个线程设置的值，而不是随机的值
* Java内存模型要求变量的读操作、写操作必须是原子性的，但对于非volatile的long、double变量，JVM允许将64位的读/写操作分解为两个32为操作

### 加锁与可见性
* 访问某个共享变量时要求所有线程在同一个锁上同步，为了确保某个线程写入该变量的值对于其它线程来说都是可见的
* 加锁的含义不仅局限于互斥，还包括内存可见性

### volatile变量
* 弱同步机制，比synchronized轻量级，确保变量的更新操作通知其它线程
* 当变量声明为volatile后，编译器与运行时都不会将该变量上的操作与其它内存操作一起重排，也不会被缓存在寄存器或对其它处理器不可见的地方，读取volatile变量时总是返回最新写入的值
* 不建议过度依赖volatile变量的可见性，这种可见性比锁更脆弱，更难以理解
* volatile变量通常做为某个操作完成、发生中断、状态标志
* 加锁机制既可以保证可见性又可以保证原子性，volatile变量只能确保可见性

## 发布与溢出
* 发布，一个对象能够在当前作用域之外的代码中使用
* 逸出，当某个不应该发布的对象被发布时
* 变量发布：static变量、getter方法、public变量、内部类

## 线程封闭
* 线程封闭，在单线程内访问数据，即某个对象封闭在一个线程中，即使被封闭的对象不是线程安全的，也不需要同步
* 线程封闭是在程序设计中的一个考虑因素，必须在程序中实现，确保封闭在线程中的对象不会从线程中逸出

### ad-hoc线程封闭
* 指维护线程封闭性的职责完全由程序实现来承担
* 单线程提供的简单性要胜过ad-hoc线程封闭技术的脆弱性
* volatile线程封闭性，只要能确保单线程对共享volatile变量执行写入操作，就可以安全的在这个共享的volatile变量上执行“读-改-写”操作，并且还能确保volatile变量对其它线程的可见性

### 栈封闭
* 指只能通过局部变量才能访问对象，局部变量固有的属性之一就是封闭在执行的线程中，它们位于执行线程的栈中，其它线程无法访问
* 对象的逸出会破坏线程封闭性

### ThreadLocal类

## 不变性
* 某个对象在被创建后其状态就不能被修改，那么这个对象就称为不可变对象，不可变对象一定是线程安全的，Java语言规范和JVM内存模型都没有给出不可变性的正式定义
* 对象不可变
	* 对象创建以后其状态不能修改
	* 对象所有域都是final类型
	* 对象创建期间，this引用没有逸出

### Fianl域

###　使用volatile类型发布不可变对象
* 通过使用包含多个状态变量的容器对象来维持不变性条件，并使用volatile类型的引用确保可见性，在没有显示锁的情况下也可保证线程安全

## 安全发布
* 多线程实例化一个对象
	* 当前线程实例化了对象，其它线程却看不到
	* 当前线程修改了对象，其它线程看到的是一个失效的值
* 可变对象的发布和使用都必须使用同步，要安全的发布一个对象，对象的引用及对象的状态必须同时对其它线程可见
* 安全发布对象的模式
	* 将对象放入线程安全的容器（Vector、Hashtable、CopyOnWriteArrayList、BlockingQueue）
	* 静态初始化器由JVM在类的初始化阶段执行，在JVM内部存在着同步机制
* 可变对象不仅在发布时需要使用同步，在每次对象访问时同样需要使用同步来确保后续修改操作的可见性

# 第四章 对象的组合
## 实力封闭
* 封装提供了一宗实例封闭机制，实例封闭机制与加锁策略结合起来，可以确保以线程安全的方式来使用非线程安全的对象
* Collections.synchronizedXXX()通过静态工厂方法，使用装饰器模式将非线程安全的容器封装在一个同步的包装器内，包装器将接口中的每个方法都实现为同步方法，并将调用请求转发到底层容器的对象上

## 线程安全性的委托

## 在现有的线程安全类中添加功能
* 重用能降低开发工作量、开发风险以及维护成本

# 第五章 基础构建模块
* 委托是创建线程安全类的一个最有效策略，让现有的线程安全类管理所有的状态即可

## 同步容器类
* 包括Vector、Hashtable等，这些同步的容器由Collections.synchronizedXXX等工厂方法创建，将它们的状态封装起来，并对每个公有方法进行同步，每次只有一个线程访问容器
* 同步容器将所有对容器的状态的访问串行化

### 同步容器类的问题
* 同步容器类都是线程安全的，但在某些情况下可能需要额外的客户端加锁来保证符合操作的安全性

		//线程不完全
		for(int i=0;i<vector.size();i++){
			doSomething(vector.get(i));
		}
		//线程安全
		synchronized(vector){
			for(int i=0;i<vector.size();i++){
				doSomething(vector.get(i));
			}
		}

### 迭代器与ConcurrentModificationException
* 当容器在迭代过程中被修改，就会抛出ConcurrentModificationException，多个线程并发的修改容器，即使是使用迭代器也无法别勉在迭代期间对容器进行加锁
* 将计数器的变化与容器关联起来，如果迭代期间计数器被修改，那么hasNext或next将抛出ConcurrentModificationException，这种检查是在没有同步的情况下进行的

### 隐藏迭代器
* 对容器的toString、equals、hashCode可能会间接的执行迭代操作，可能会抛出ConcurrentModificationException

## 并发容器
* Java5.0提供并发容器来改进同步容器的性能，并发容器针对多个线程并发访问设计，并发容器可以提高伸缩性并降低风险

### ConcurrentHashMap
* ConcurrentHashMap并不是将每个方法都在同一个锁上同步并使得每次只有一个线程访问容器，而是采用分段锁机制，一种粒度更细的加锁机制实现更大程度的共享，在并发环境下将实现更高的吞吐量，而在单线程环境中只损耗非常小的性能
* ConcurrentHashMap返回的迭代器具有弱一致性，并非“及时失败”，以容忍并发的修改
* ConcurrentHashMap中的size、isEmpty的准确性被略微弱化，返回的结果在可能已经过期，它实际上只是一个估值
* 只有当应用程序需要加锁的Map以进行独占访问时，才应该弃用ConcurrentHashMap

### 额外的原子操作
* Remove-If-Equal、Replace-If-Equal、Put-If-Absent这些原子性操作在ConcurrentHashMap中都有对应的接口实现了原子性操作

### CopyOnWriteArrayList
* 每次修改时都会创建并重新发布一个新的容器副本，从而实现可变性
* 每当修改容器时都会复制底层数组，这需要一定的开销，仅当迭代操作远远多余修改操作时，才应该使用“写入时复制”容器

## 阻塞队列和生产者-消费者模式
* 如果队列已经满，put方法将阻塞直到有空间可用，如果队列为空，take方法将阻塞直到有元素可用
* BlockingQueue
	* LinkedBlockingQueue
	* ArrayBlockingQueue
	* PriorityBlockingQueue
	* SynchronousQueue

## 阻塞方法与中断
* 当某个方法抛出InterruptedException时，表示该方法是一个阻塞方法，如果这个方法被中断，那么它将努力提前结束阻塞状态
* InterruptedException处理方式
	* 传递InterruptedException抛给调用者处理
	* InterruptedException会清空中断标记，在catch中重新设置中断标记

## 同步工具类
* 同步工具类包含以下特殊的工具属性，这些属性封装了一些状态，这些状态决定同步工具类的线程何时继续执行还是等待

### 闭锁
* 闭锁可以延迟线程的进度，作用相当于一扇门，在闭锁到达结束状态前，这扇门一直是关着的，没有任何线程能通过，当到达结束状态时，这扇门就会被打开，允许线程通过
* 当闭锁到达结束状态，将不会在改变状态，因此这扇门将永远保持打开状态

### FutureTask
* 可以用作闭锁，表示计算是通过Callable来实现的，相当于一种可生成结果的Runable
* 如果任务完成get立刻返回结果，否则get将阻塞直到任务进入完成状态
* FutureTask将计算结果从执行计算的线程传递到获取结果的线程，这个传递过程时安全发布的

### 信号量
* 计数信号量:用来控制同时访问某个特定资源的操作数量，或者同时执行某个指定操作的数量
* 二值信号量，Semaphore初始值为1，可以用作互斥锁，并且不可重入，谁拥有了唯一的许可，谁就拥有了互斥锁

### 栅栏
* 阻塞一组线程直到某个事件发生，与闭锁的区别在于，所有线程必须同时到达栅栏位置，才能继续执行
* 闭锁用于等待事件，栅栏用于等待其它线程
* 当线程到达栅栏位置时将调用await()阻塞当前线程，直到所有线程都到达栅栏位置，所有线程都到达栅栏位置时，栅栏讲打开，所有线程被释放，而栅栏讲被重置以便下次使用

# 第六章 任务执行
## 在线程中执行任务
* 任务之间应该相互独立，不依赖于其它任务的状态、结果或边界效应

### 串行的执行任务
* 在单线程的服务其中，阻塞不仅会推迟当前请求完成的时间，还将彻底阻止等待中的请求被处理

### 显示地为任务创建线程
* 为每个任务单独的创建一个线程，能提升串行执行的性能
* 没有限制的创建线程的数量，会消耗系统资源

## Executor框架
* 任务是一组逻辑工作单元，而线程则是使任务异步执行的机制，Java类库中任务的主要抽象不是Thread，而是Executor
* Executor基于生产者-消费者模式，提交任务的操作相当于生产者，执行任务的相当于消费者
* 希望获得更灵活的执行策略时，考虑用Executor代替Thread

### 线程池
* 线程池与工作队列密切相关，队列中保存了所有等待执行的任务，工作者线程从工作队列中获取一个任务，执行任务，然后返回线程池并等待下一个任务
* 重用现有线程而不是创建新的线程，可以在处理多个请求时分摊在线程创建和销毁过程中产生的巨大开销
* Executor
	* newFixedThreadPool，创建一个固定长度的线程池，每当提交一个任务时就创建一个线程，直到到达线程池的最大数量（如果线程由于未预期的Exception而结束，那么线程池会补充一个新的线程）
	* newCachedThreadPool，创建一个缓存的线程池，线程池当前线程数量超过了处理需求时，将回收空闲线程，而当需求线程增加时，将创建新的线程，线程池规模不存在任何限制
	* newSingleThreadPool，单线程的Excecutor，创建单个线程，如果这个线程异常结束，将创建新的线程
	* newScheduledThreadPool，创建一个固定长度的线程池，以延迟或定时的方式执行任务

### Executor的生命周期
* ExecutorService扩展了Executor接口，添加了一些用于生命周期管理的方法
* ExecutorService生命周期
	* 运行，初始创建时
	* 关闭，shutdown方法，将平缓的关闭（不再接受新的任务，同时等待已经提交的任务执行完成）
	* 终止，shutdownNow方法，尝试取消所有运行中的任务，并且不会再启动队列中尚未开始执行的任务

### 延迟任务与周期任务
* Timer支持基于绝对时间而不是相对时间的调度机制，因此任务执行时对系统时钟变化敏感，newScheduledThreadPool只支持基于相对时间的调度
* Timer在执行所有定时任务时只会创建一个线程
* 线程泄漏，Timer线程不捕获异常，当TimerTask抛出未检查的异常时将终止定时线程，Timer也不会恢复线程的执行，而是错误的认为整个Timer都被取消，已经被调度但尚未执行的TimerTask将不会再执行，新任务也不能被调度

## 找出可利用的并行性
### Callable与Future
* Callable相对Runnable是一种更好的抽象，它以call()为入口，将返回一个值，并可能抛出一个异常
* Future表示一个任务的生命周期，并提供了相应的方法来判断是否已经完成、取消、获取任务的结果等

### CompletionService
* CompletionService将Executor和BlockingQueue功能融合在一起

		private static final ExecutorService threadPool = Executors.newFixedThreadPool(10);
		CompletionService<String> cs = new ExecutorCompletionService<String>(threadPool);
		for (int i = 0; i < 100; i++) {
			cs.submit(new Index(String.valueOf(i)));
		}
		for (int i = 0; i < 100; i++) {
			Future<String> future = cs.take();
			String result = future.get();
			System.out.println("i="+result);
		}


# 第七章 取消与关闭
* Java没有提供任何机制来安全地终止线程，但它提供了中断这种协作机制，能够使一个线程终止另一个线程的当前工作
* 行为良好的软件能很完善地处理失败、关闭、取消等过程

## 任务取消
* Java中没有一种安全的抢占方法来停止线程，因此就没有安全的抢占方式来停止任务（每个线程都是抽象的任务）

### 中断
* 线程中断是一种协作机制，线程可以通过这个机制通知另一个线程，告诉它在合适的情况下停止当前工作，并转而执行其它工作，如果在取消之外的其它操作中使用中断，都是不合适的
* 如果任务代码能够响应中断状态，可以使用中断作为取消机制，中断是实现取消最合理的方式
* 阻塞方法Thread.sleep、Object.wait等都会检查线程何时中断，并在发现中断前提前返回
* 响应中断操作包括
	1. 清除中断标记
	2. 抛出InterruptedException（表示阻塞操作由于中断而提前结束）
* JVM不保证阻塞方法检测中断的速度，但实际情况速度还是非常快的
* 调用interrupt并不意味着立即停止目标线程正在进行的工作，而只是传递了请求中断消息，由线程在一个合适的时刻中断自己

		@Override
		public void run() {
			BigInteger p = BigInteger.ONE;
			while (!Thread.currentThread().isInterrupted()) {
				p = p.nextProbablePrime();
				System.out.println("task is running。。。"+p);
				try {
					queue.put(p);
					TimeUnit.SECONDS.sleep(1);
				} catch (InterruptedException e) {
					//TODO 线程退出逻辑
					System.out.println(Thread.currentThread().getName() + " 中断");
				}
			}
		}

### 中断策略
* 中断策略，尽快退出执行流程，在必要时进行清理，并把中断信息传递给调用者，从而使调用栈中的上层代码可以采取进一步的操作
* 无论任务把中断视为取消，还是其它某个中断响应操作，都应该小心保存执行线程的中断中断状态，如果除了将InterruptedException传递给调用者外还需要其它操作，应该在捕获InterruptedException之后恢复中断状态
* 每个线程都拥有各自的中断策略，除非你知道中断对该线程的含义，否则就不应该中断这个线程

### 响应中断
* 当ThreadPoolExecutor拥有的工作者线程检测到中断时，它会检查线程池是否正常关闭，故如果是，它会在结束前执行一些线程池清理工作，否则它可能创建一个新的线程将线程池恢复到合理的规模

### 通过Future来实现取消
    public static void timedRun(Runnable r，long timeout， TimeUnit unit)throws InterruptedException {
        Future<?> task = taskExec.submit(r);
        try {
            task.get(timeout， unit);
        } catch (TimeoutException e) {
            // task will be cancelled below
        } catch (ExecutionException e) {
            // exception thrown in task; rethrow
            throw launderThrowable(e.getCause());
        } finally {
            // Harmless if task already completed
            task.cancel(true); // interrupt if running
        }
    }

## 停止基于线程的服务
* 线程封装的原则，除非拥有某个线程，否则不能对该线程进行操控
* 应用程序可以拥有服务，服务可以用拥有工作者线程，但应用程序不能拥有工作者线程

# 第八章 线程池的使用

## 在任务与执行策略之间的隐形耦合

### 线程饥饿死锁
* 单线程的Executor中，如果一个任务将另一个任务提交到同一个Executor，并且等待这个被提交的任务结果，通常会发生死锁，第二个任务停留在工作队列中，并等待第一个任务完成，而第一个任务又无法完成，在等待第二个任务完成
* 饥饿死锁：只要线程池中的任务需要无限期的等待一些必须由池中其它任务才能提供的资源或条件，例如某个任务等待另一个任务的返回值或执行结果
* 每当体提交一个有依赖的Executor任务时，要清楚的知道可能会出现的线程“饥饿”死锁

### 运行时间较长的任务
* 执行时间较长的任务不仅会造成线程池阻塞，还会增加执行时间较短任务的服务时间
* 限定任务等待资源的时间，如果等待超时，可以把任务标识为失败，然后中止任务或者将任务重新放回队列以便随后执行
* 如果在线程池中总是充满了被阻塞的任务，那么也可能表明线程池的规模过小

## 设置线程池大小
* 线程池过大，大量的线程将在相对很少的CPU和内存资源上发生竞争，会导致更高的内存使用量，还可能耗尽资源
* 线程池过小，将导致许多空闲的处理器无法执行工作，从而降低吞吐量
* 密集型的任务cpu个数+1时，通常能实现最优的利用率，I/O操作或阻塞操作的任务，由于线程并不会一定执行，因此线程池规模应该更大

## 配置ThreadPoolExecutor

    public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit uni,
															BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler)

### 线程的创建与销毁
* corePoolSize：线程池目标大小，在没有任务执行时线程池的大小，并且只有在工作队列满的情况下才会创建超出这个数量的线程
* maximumPoolSize：线程池同时活动的线程数量上限
* keepAliveTime：某个线程的空闲时间超过了keepAliveTime，那么将被标记为可回收的，并且当线程池的当前大小超过corePoolSize时，这个线程将被终止

	    public static ExecutorService newCachedThreadPool() {
	        return new ThreadPoolExecutor(0， Integer.MAX_VALUE，60L， TimeUnit.SECONDS，new SynchronousQueue<Runnable>());
	    }

	    public static ExecutorService newFixedThreadPool(int nThreads) {
	        return new ThreadPoolExecutor(nThreads， nThreads，0L， TimeUnit.MILLISECONDS，new LinkedBlockingQueue<Runnable>());
	    }

### 管理队列任务
* 通过采用固定大小的线程池来解决无限制的创建线程问题，但如果请求的到达速率超过了线程池的处理速率，请求会在一个有Executor管理的Runnable队列中等待，不会去竞争CPU资源
* newFixedThreadPool、newSingleThreadPool，在默认情况下将使用一个无界的LinkedBlockingQueue，如果工作者线程都处于忙碌状态，任务将在队列中等候，如果任务持续的到达，并且超过了线程池处理它们的速度，队列将无限制的增加
* newCachedThreadPool使用SynchronousQueue，SynchronousQueue避免任务排队，直接将任务从生产者移交给消费者，SynchronousQueue不是一个真正的队列，而是一种在线程之间进行移交的机制，如果没有消费者线程正在等待，并且线程池大小小于maximumPoolSize，那么ThreadPoolExecutor将创建一个新的线程，否则根据饱和策略拒绝这个任务

### 饱和策略
* JDK提供RejectedExecutionHandler实现
	* AbortPolicy:默认策略，抛出未检查的RejectedExecutionException
	* DiscardPolicy:抛弃任务
	* DiscardOldestPolicy:抛弃下一个将被执行的任务，并尝试重新提交新的任务，如果和优先级队列一起使用，将抛弃优先级最后的任务
	* CallerRunsPolicy:不抛异常，不抛弃任务，讲任务回退给调用者，在调用execute的线程中执行该任务

### 线程工厂方法
* 每当线程池需要创建一个线程时，通过线程工程方法完成，默认的线程工厂方法将创建一个新的、非守护的线程，并且不包含特殊的配置信息

		public interface ThreadFactory {
		    Thread newThread(Runnable r);
		}

## 扩展ThreadPoolExecutor
* ThreadPoolExecutor生命周期方法
	* protected void beforeExecute(Thread t， Runnable r)，在线程运行前执行，如果beforeExecute抛出RuntimeException，那么任务将不被执行，并且afterExecute()也不会被执行
	* protected void afterExecute(Runnable r， Throwable t)，在线程运行之后执行，无论run()正常返回，还是抛出异常返回，afterExecute()都会被调用，如果线程完成后带有一个Error，那么afterExecute()将不会被调用
	* protected void terminated(),在线程池完成关闭操作时调用
