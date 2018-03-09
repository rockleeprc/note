

## 基础概念

### 同步 Synchronous/异步 Asynchronous

同步方法：调用一旦开始，调用者必须等到方法调用返回后，才能继续后续行为

异步方法：调用一旦开始，立即返回，调用者可以继续后续的操作，如需返回结果，当异步调用真实完成时，会通知调用者

### 并发 Concurrency/并行 Parallelism

并发：偏重于多个任务交替执行，多任务间有可能还是串行

并行：真正意义上的任务同时执行

一个cpu一次只能执行一条指令，这种情况下多进程、多线程就是并发的，不是并行，操作系统会不停的切换任务，真实的并行出现在拥有多核cpu的情况下

### 临界区

临界区用来表示公共资源、共享数据，可以被多个线程访问，但每一次，只能有一个线程使用，临界区资源被占用，其他线程想要使用临界区资源，只能等待

### 阻塞 Blocking/非阻塞 Non-Blocking

阻塞、非阻塞用来形容多线程间的互相影响

阻塞：一个线程占用临界区资源，其他需要该资源的线程必须等待，等待导致线程挂起

非阻塞：没有一个线程可以妨碍其他线程

### 线程活跃性

死锁 Deadlock：A任务等待B任务，B任务等待C任务，C任务等待A任务

饥饿 Starvation：一个或多个线程因其他原因无法获得所需资源，导致一直无法执行

	1. 线程优先级太低，而高优先级的线程不断抢占资源，倒入地优先级线程无法工作
	2. 某一个线程一直占用着资源不释放，其他需要这个资源的线程无法正常执行

活锁 Livelock：A线程主动将资源释放B线程，B线程获得资源后主动释放给A线程，资源不断的带A、B两个线程间跳动

## 并发级别

### 阻塞 Blocking

一个线程是阻塞的，在其他线程释放资源前，当前线程无法继续执行

synchronized、重入锁都会在访问临界区资源前得到临界区的锁，如果得不到，线程会被挂起等待，直到获得锁

### 无饥饿 Starvation-Free

非公平锁：允许高优先级的线程插队，可能导致饥饿

公平锁：满足先来后到，不会产生饥饿

### 无障碍 Obstruction-Free

最弱的非阻塞调度机制，允许多个线程在没有锁的情况下进入临界区，当多个线程修改共享资源时，一旦检测到，线程会立刻对自己所做的修改进行回滚

当临界区内存在严重冲突时，所有线程可鞥都会不断地回滚自己的操作，没有一个线程可以走出临界区

### 无锁 Lock-Free

无锁的并行都是无障碍，所有线程都尝试对临界区进行访问，无锁保证必然有一个线程能够在有限步内完成操作离开临界区，其他线程需要回滚

在无障碍、无锁中，当线程不断尝试修改，不断回滚，则会出现饥饿现象

### 无等待 Lock-Free

在无锁的基础上扩展，要求所有线程都必须在有限步内完成，避免出现饥饿现象

## 线程安全性

### 原子性 Atomicity

一个线程对共享变量进行读/写操作，在其它线程看来该操作要么执行结束，要么从未发生

	static int i;
	A线程 i=1;
	B线程i=-1;
不管两个线程以何种方式，何种步调工作，i的值要么是1,要么是-1

32位系统中long类型的读写不是原子性的，因为long占用64位空间，两个线程同时对long进行读/写操作，线程间的结果有干扰的可能发生

### 可见性 Visibility

一个线程对共享变量进行操作，后续访问该变量的线程可能无法立刻读取到最新的结果，甚至永远无法读取到

### 有序性 Ordering

程序执行时，可能会进行指令重排，重排的指令与原指令的顺序未必一直

指令重排可以保证串行语义一致，但不能保证多线程语义也一致

#### Hapen-Before原则

1. 一个线程内保证语义串行性
2. volatile变量的写先于读，保证变量可见性
3. unlock先于lock
4. A先于B，B先于C，A一定先于C
5. Thead.start()先于Thread.run()
6.
7.
8. 对象构造函数执行先于finalize()
it
## 线程同步机制

## 线程状态

	public enum State {
	  /*刚刚创建的线程，还没开始执行，离开NEW状态后将不能在回到NEW状态*/
		NEW,
		/*调用start()*/
		RUNNABLE,
		/*在RUNNABLE状态中，遇到了synchronized，会进入到BLOCKED中，暂停线程执行，直到获取请求的锁*/      
		BLOCKED,
   		/*进无时间限制的等待，wait()需要notify()唤醒，join()等待线程目标线程的终止*/
		WAITING,
	   	/*进有时间限制的等待，wait()需要notify()唤醒，join()等待线程目标线程的终止*/   
		TIMED_WAITING,
   		/*线程执行结束，处于TERMINATED状态的线程不能在回到其它状态*/
		TERMINATED;
  	}

## 线程基本操作

### start()

	Thread t = new Thread();
	t.start();

	Thread t = new Thread(new Runnable());
	/*调用Runnable.run()，见Thread.run()源码*/
	t.start();

### stop()

	Thread t = new Thread();
	t.start();
	do something...
	t.stop();

Thread.stop()直接终止线程，立即释放该线程持有的锁，当线程执行到一半时，这种强行终止线程会导致数据不一致的发生

### interrupt()

线程中断并不会立刻退出，只是给线程发送一个通知，告诉目标线程，有人希望你退出，目标线程接到通知后如何处理，是由目标线程自行决定

	public class Thread implements Runnable{
		/*设置中断标志位*/
		public void interrupt();
		/*判断当前线程是否被中断*/
		public boolean isInterrupted();
		/*判断线程的中断状态，清除当前线程的中断标志位*/
		public static boolean interrupted();
	}

当使用Thread.sleep()时会抛出InterruptedExeption，此时线程的中断标记会被清除，如果需要中断 标记，在catch{}中进行重置

### wait()/notify()
	public class Object {
		public final void wait() throws InterruptedException;
		public final native void notify();
	}

在当前对象上调用wait()后，当前线程会在该对象上等待，并且释放锁（slepp不释放资源），直到其他线程调用notify()

一个线程调用Object.wait()后，当前线程对象会进入Object对象的等待队列，Object.notify()被调用后，会从等待队列中随机选择一个线程唤醒，并不是先等待的线程优先被唤醒

notifyAll()功能与notify()一样，但会唤醒等待队列中所有的线程，不是随机选择一个

wait()/notify()在使用前都需要获取锁，所以必须在synchronized中调用

### suspend()/resume()

被suspend()的线程，必须要等到resume()后才能继续执行，suspend()使线程暂停的同时并不会释放任何资源，被挂起的线程状态仍然是RUNNABLE

对于suspend()和resume()的行为可以使用wait()和notify()实现


### join()/yield()

当前线程的输入可能依赖于另外一个或多个线程的输出，此时当前线程就需要等待依赖的线程执行完毕后才能继续运行

join()表示无限期等待，一直阻塞当前线程，直到目标线程执行完成

yield()当前线程一些重要的工作已经完成，可以休息一下，给其它线程一些执行机会

### daemon()

一个Java应用程序内，只有守护线程时，Java虚拟机会自动退出

	t.setDaemon(true);
	t.start();

	Exception in thread "main" java.lang.IllegalThreadStateException

setDaemon(true)一定要在start()前，不然会得到一个IllegalThreadStateException异常，setDaemon()设置失败，但程序仍能正常执行

### synchronized

synchronized (instance){} 对给定对象进行加锁，进入同步代码前要获取instance对象的锁

public void synchronized method(){} 对当前实例进行加锁，进入同步方法前需要获取当前实例的锁

public static synchronized void method(){} 对当前类进行加锁，进入同步方法前需要获取当前类的锁

### volatile

volatile不能保证线程安全，它只能确保一个线程修改数据后，其它线程能够看到这个改动

volatile仅能保证写操作的原子性，不能保证read-modify-write/check-then-act的原子性

## 同步控制

### ReetrantLock

	public class ReentrantLock implements Lock, java.io.Serializable {
		/*true，公平锁，公平锁实现必须依靠系统维护一个有序的队列，实现成本高，性能相对较低
			false，非公平锁，在锁的等待队列中随机挑选一个线程，会产生饥饿*/
		public ReentrantLock(boolean fair)
		/*获取锁，如果锁被占用则等待*/
		public void lock();
		/*释放锁*/
		public void unlock();
		/*尝试获取锁，如果成功返回true，失败返回false，不会等待，立即返回*/
	 	public boolean tryLock();
		/*在给定时间内尝试获取锁*/
		public boolean tryLock(long timeout, TimeUnit unit) throws InterruptedException;
		/*获取锁，但优先响应中断*/
		public void lockInterruptibly() throws InterruptedException;
		/*返回一个绑定到此 Lock 实例上的 Condition 实例*/
		public Condition newCondition();
	}


### Condition

	public interface Condition {
		/*当前线程等待，释放锁*/
		void await() throws InterruptedException;
		/*和await()相同，但不会在等待过程中响应中断*/
		void awaitUninterruptibly();
		/*唤醒一个等待的中的线程*/
		void signal()
	}

### Semaphore

synchronized、ReentrantLock一次都只能允许一个线程访问一个资源，Semaphore可以指定多个线程同时访问一个资源，在执行操作前首先要获取许可，并在使用后释放许可

### ReadWriteLock

读写分离的锁

### CountDownLatch

控制线程等待，可以让一个线程等待直到倒计时结束，再开始执行

### CyclieBarrier

当线程到达栅栏位置时将调用await()将线程阻塞，直到所有线程都到达栅栏，所有线程到达栅栏位置后，栅栏将打开，释放所有线程，栅栏被重置，以便下次继续使用

### LockSupport

线程阻塞工具类，在线程内任意位置让线程阻塞，不需要获得锁，不会抛出InterruptedException


## 线程池

### Executors
	public class Executors {
		/*固定数量的线程池，一个新任务提交时，线程池有空闲线程，立即执行，没有空闲线程，任务会被放到一个任务队列中，
		等有空间线程时在执行*/
		public static ExecutorService newFixedThreadPool(int nThreads);
		/*只有一个线程，当多余一个任务时，任务讲被保存在任务队列中*/
		public static ExecutorService newSingleThreadExecutor();
		/*根据实际情况调整线程数量，线程池中的线程数量不确定，当有空间线程时优先使用空闲线程，
		当没有空闲线程，又有新任务提交时，创建新线程执行*/
		public static ExecutorService newCachedThreadPool();
		/*一个线程，在设定的时间延时后执行任务*/
		public static ScheduledExecutorService newSingleThreadScheduledExecutor();
		/*可指定线程池大小，在设定的对任务进行调度*/
		public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize)
	}

### ScheduledExecutorService
调度程序不保证任务会无限期的持续调度，如果任务遇到异常，那么后续的所有任务讲都会停止执行


	public interface ScheduledExecutorService extends ExecutorService {
		/*在给定的时间对任务调度一次*/
		public ScheduledFuture<?> schedule(Runnable command,long delay, TimeUnit unit);
		/*以上一个任务开始执行时间为起点，之后的period时间调度下一次任务，
		后续的第一任务在initialDelay+period时执行，
		后续的第二任务在initialDelay + 2 * period时执行，一次类推*/
		public ScheduledFuture<?> scheduleAtFixedRate(Runnable command,long initialDelay,long period,TimeUnit unit);
		/*在上一个任务结束后，经过delay时间，在调度*/
		public ScheduledFuture<?> scheduleWithFixedDelay(Runnable command,long initialDelay,long delay,TimeUnit unit);
	}

### ThreadPoolExecutor

	public ThreadPoolExecutor(int corePoolSize,/*线程池中的线程数量*/
				int maximumPoolSize,/*线程池中最大线程数*/
				long keepAliveTime,/*线程池数量超过corePoolSize时，多余的空闲线程的存活时间*/
				TimeUnit unit,/*keepAliveTime单位*/
				BlockingQueue<Runnable> workQueue,/*线程被提交，但未被执行的任务队列*/
				ThreadFactory threadFactory,/*线程工厂，用于创建线程*/
				RejectedExecutionHandler handler/*当任务太多来不及处理时的拒绝策略*/
				)

* 任务提交
	* 小于corePoolSize，分配线程执行
	* 大于corePoolSize，提交到等待队列
		* 成功，等待执行
		* 失败，提交线程池
			* 大于maximumPoolSize，拒绝策略
			* 小于maximumPoolSize，分配线程执行

### 拒绝策略

* AbortPolicy，直接抛出异常，组织任务正常工作
* CallerRunsPolicy，只要线程池为关闭，直接在调用者线程中运行任务
* DiscardPolicy，默默丢弃无法处理的任务，不予任何处理
* DiscardOldestPolicy，丢弃最老的一个请求，再次尝试提交当前任务

## 并发容器

### CopyOnWriteArrayList

### BlockingQueue



### SkipList

## “锁”优化

性能优化时根据运行时的真实情况对各个资源点进行权衡这种的过程

### 减小锁时间

如果线程持有锁时间越长，锁的竞争程度就越激烈，减小锁的持有时间有助于降低锁冲突的可能性，进而提高并发能力

	/*整个方法都被加锁*/
	public synchronized void synMethod(){
		...
		doSomething();/*重量级方法*/
		mutexMethod();
		doSomething();/*重量级方法*/
		...
	}

	public void synMethod(){
		...
		doSomething();
		/*只在必要时进行同步，减小线程持有锁的时间*/
		synchronized(this){
			mutexMethod();
		}
		doSomething();
		...
	}

### 减小锁粒度

缩小锁对象的范围，从而减少锁冲突的可能性，进而提高并发能力

参见ConcurrentHashMap.java

### 读写分离锁替换独占锁

在读多写少的场合，读写分离锁效率更高

### 锁分离

参考LinkedBlockingQueue.java实现

### 锁粗化

JVM遇到连续地对同一把锁不断的请求和释放操作时，会把所有的锁操作整合成对锁的一次请求，从而减少对锁的请求同步次数

## ThreadLoacl

## CAS

CAS(V,E,N) 仅当V.value==E时，才会将V.setValue(N)
