## 什么时候使用线程池
1. 单个任务处理时间比较短：程序的快速响应
2. 需要处理的任务数量很大：增加程序吞吐量


## 线程执行策略
```java
public void execute(Runnable command) {
    if (command == null) throw new NullPointerException();
    int c = ctl.get(); //clt记录着runState和workerCount
    /*
     * workerCountOf方法取出低29位的值，表示当前活动的线程数；
     * 如果当前活动线程数小于corePoolSize，则新建一个线程放入线程池中，并把任务添加到该线程中。
     */
    if (workerCountOf(c) < corePoolSize) {
        /*
         * addWorker中的第二个参数表示限制添加线程的数量是根据corePoolSize来判断还是maximumPoolSize来判断；
         * 如果为true，根据corePoolSize来判断；如果为false，则根据maximumPoolSize来判断
         */
        if (addWorker(command, true)) return;
        c = ctl.get(); // addWorker()添加失败，则重新获取ctl值
    }

    // 如果当前线程池是运行状态并且任务添加到队列成功
    if (isRunning(c) && workQueue.offer(command)) {
        int recheck = ctl.get(); // 重新获取ctl值
        // 再次判断线程池的运行状态，如果不是运行状态，由于之前已经把command添加到workQueue中了，这时需要移除该command
        if (!isRunning(recheck) && remove(command))
            reject(command); // 通过handler使用拒绝策略对该任务进行处理，整个方法返回   
        else if (workerCountOf(recheck) == 0) //获取线程池中的有效线程数，如果数量是0，则执行addWorker方法
            /*
             * 这里传入的参数表示：
             * 1. 第一个参数为null，表示在线程池中创建一个线程，但不去启动；
             * 2. 第二个参数为false，将线程池的有限线程数量的上限设置为maximumPoolSize，添加线程时根据maximumPoolSize来判断；
             * 如果判断workerCount大于0，则直接返回，在workQueue中新增的command会在将来的某个时刻被执行。
             */
            addWorker(null, false);
    } 
    // 再次调用addWorker方法，但第二个参数传入为false，将线程池的有限线程数量的上限设置为maximumPoolSize；
    else if (!addWorker(command, false)) 
        /*
         * 如果执行到这里，有两种情况：
         * 1. 线程池已经不是RUNNING状态；
         * 2. 线程池是RUNNING状态，但workerCount >= corePoolSize并且workQueue已满。
         */
        reject(command); // 如果失败则拒绝该任务
}
```
* 以上代码对应的执行逻辑图示
![avatar](../img/thread_pool_execute.jpg)

## 线程池中创建线程并执行

```java
private boolean addWorker(Runnable firstTask, boolean core) {
    retry:
    for (;;) {
        int c = ctl.get(); //clt记录着runState和workerCount
        int rs = runStateOf(c); // 获取运行状态
        
        /*
         * 这个if判断
         * 如果rs >= SHUTDOWN，则表示此时不再接收新任务；
         * 接着判断以下3个条件，只要有1个不满足，则返回false：
         * 1. rs == SHUTDOWN，这时表示关闭状态，不再接受新提交的任务，但却可以继续处理阻塞队列中已保存的任务
         * 2. firsTask为空
         * 3. 阻塞队列不为空
         * 
         * rs == SHUTDOWN的情况
         * 这种情况下不会接受新提交的任务，所以在firstTask不为空的时候会返回false；
         * firstTask为空，并且workQueue也为空，则返回false，
         */
        // Check if queue empty only if necessary.
        if (rs >= SHUTDOWN && ! (rs == SHUTDOWN && firstTask == null && ! workQueue.isEmpty()))
            return false; // 不添加线程返回false
        
        for (;;) {
            int wc = workerCountOf(c); // 获取线程数
            /*
             * 如果wc超过CAPACITY，也就是ctl的低29位的最大值（二进制是29个1），返回false；
             * 这里的core是addWorker方法的第二个参数，如果为true表示根据corePoolSize来比较，
             * 如果为false则根据maximumPoolSize来比较。
             */
            if (wc >= CAPACITY || wc >= (core ? corePoolSize : maximumPoolSize))
                return false;
            if (compareAndIncrementWorkerCount(c)) // 尝试增加workerCount，如果成功，则跳出第一个for循环
                break retry;
            c = ctl.get();  // Re-read ctl // 如果增加workerCount失败，则重新获取ctl的值
            if (runStateOf(c) != rs) // 如果当前的运行状态不等于rs，说明线程池状态已被改变，返回第一个for循环继续执行
                continue retry;
            // else CAS failed due to workerCount change; retry inner loop
        }
    }
    boolean workerStarted = false;
    boolean workerAdded = false;
    Worker w = null;
    try {
        w = new Worker(firstTask); // 根据firstTask来创建Worker对象
        final Thread t = w.thread; // 每一个Worker对象都会创建一个线程
        if (t != null) {
            final ReentrantLock mainLock = this.mainLock;
            mainLock.lock();
            try {
                // Recheck while holding lock.
                // Back out on ThreadFactory failure or if
                // shut down before lock acquired.
                int rs = runStateOf(ctl.get());
                // rs < SHUTDOWN表示是RUNNING状态；
                // 如果rs是RUNNING状态或者rs是SHUTDOWN状态并且firstTask为null，向线程池中添加线程。
                // 因为在SHUTDOWN时不会在添加新的任务，但还是会执行workQueue中的任务
                if (rs < SHUTDOWN || (rs == SHUTDOWN && firstTask == null)) {
                    if (t.isAlive()) // precheck that t is startable
                        throw new IllegalThreadStateException();
                    workers.add(w); // workers是一个HashSet
                    int s = workers.size();
                    // largestPoolSize记录着线程池中出现过的最大线程数量
                    if (s > largestPoolSize) largestPoolSize = s;
                    workerAdded = true;
                }
            } finally {
                mainLock.unlock();
            }
            if (workerAdded) {
                t.start(); // 启动线程
                workerStarted = true;
            }
        }
    } finally {
        if (! workerStarted) addWorkerFailed(w);
    }
    return workerStarted;
}
```

## Worker中线程的执行
```java
final void runWorker(Worker w) {
    Thread wt = Thread.currentThread();
    Runnable task = w.firstTask; // 获取第一个任务
    w.firstTask = null;
    w.unlock(); // allow interrupts // 允许中断
    boolean completedAbruptly = true; // 是否因为异常退出循环
    try {
        // 如果task为空，则通过getTask来获取任务
        while (task != null || (task = getTask()) != null) {
            w.lock();
            // If pool is stopping, ensure thread is interrupted; if not, ensure thread is not interrupted.  
            // This requires a recheck in second case to deal with shutdownNow race while clearing interrupt
            if ((runStateAtLeast(ctl.get(), STOP) || 
                (Thread.interrupted() && runStateAtLeast(ctl.get(), STOP))) &&
                !wt.isInterrupted())
                wt.interrupt();
            try {
                beforeExecute(wt, task);
                Throwable thrown = null;
                try {
                    task.run();
                } catch (RuntimeException x) {
                    thrown = x; throw x;
                } catch (Error x) {
                    thrown = x; throw x;
                } catch (Throwable x) {
                    thrown = x; throw new Error(x);
                } finally {
                    afterExecute(task, thrown);
                }
            } finally {
                task = null;
                w.completedTasks++;
                w.unlock();
            }
        }
        completedAbruptly = false;
    } finally {
        processWorkerExit(w, completedAbruptly);
    }
}
```

## 从阻塞队列中取任务
```java
private Runnable getTask() {
    // timeOut变量的值表示上次从阻塞队列中取任务时是否超时
    boolean timedOut = false; // Did the last poll() time out?
    for (;;) {
        int c = ctl.get();
        int rs = runStateOf(c);
        // Check if queue empty only if necessary.
        /*
         * 如果线程池状态rs >= SHUTDOWN，也就是非RUNNING状态，再进行以下判断：
         * 1. rs >= STOP，线程池是否正在stop；
         * 2. 阻塞队列是否为空。
         * 如果以上条件满足，则将workerCount减1并返回null。
         * 因为如果当前线程池状态的值是SHUTDOWN或以上时，不允许再向阻塞队列中添加任务。
         */
        if (rs >= SHUTDOWN && (rs >= STOP || workQueue.isEmpty())) {
            decrementWorkerCount();
            return null;
        }
        int wc = workerCountOf(c);
        // Are workers subject to culling?
        // timed变量用于判断是否需要进行超时控制。
        // allowCoreThreadTimeOut默认是false，也就是核心线程不允许进行超时；
        // wc > corePoolSize，表示当前线程池中的线程数量大于核心线程数量；
        // 对于超过核心线程数量的这些线程，需要进行超时控制
        boolean timed = allowCoreThreadTimeOut || wc > corePoolSize;
        
        /*
         * wc > maximumPoolSize的情况是因为可能在此方法执行阶段同时执行了setMaximumPoolSize方法；
         * timed && timedOut 如果为true，表示当前操作需要进行超时控制，并且上次从阻塞队列中获取任务发生了超时
         * 接下来判断，如果有效线程数量大于1，或者阻塞队列是空的，那么尝试将workerCount减1；
         * 如果减1失败，则返回重试。
         * 如果wc == 1时，也就说明当前线程是线程池中唯一的一个线程了。
         */
        if ((wc > maximumPoolSize || (timed && timedOut))
            && (wc > 1 || workQueue.isEmpty())) {
            if (compareAndDecrementWorkerCount(c))
                return null;
            continue;
        }
        try {
            /*
             * 根据timed来判断，如果为true，则通过阻塞队列的poll方法进行超时控制，如果在keepAliveTime时间内没有获取到任务，则返回null；
             * 否则通过take方法，如果这时队列为空，则take方法会阻塞直到队列不为空。
             */
            Runnable r = timed ? workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) : workQueue.take();
            if (r != null)
                return r;
            // 如果 r == null，说明已经超时，timedOut设置为true
            timedOut = true;
        } catch (InterruptedException retry) {
            // 如果获取任务时当前线程发生了中断，则设置timedOut为false并返回循环重试
            timedOut = false;
        }
    }
}
```

## 构造函数
```java
public ThreadPoolExecutor(int corePoolSize, // 
                            int maximumPoolSize, // 
                            long keepAliveTime, // 
                            TimeUnit unit, // keepAliveTime的时间单位
                            BlockingQueue<Runnable> workQueue, // 
                            ThreadFactory threadFactory, // 线程池工厂
                            RejectedExecutionHandler handler) // 超出maximumPoolSize + workQueue总和时，应用的拒绝策略
```
* corePoolSize：workQueue没满时，线程池最大线程数，当线程池中线程数量小于corePoolSize时，新提的交任务将创建一个新的线程执行，即使线程池中存在空闲线程，线程池数量达到corePoolSize时，新提交的任务将进入workQueue中，等待线程池调度执行
* maximumPoolSize：workQueue满时，线程池最大线程数，
* workQueue：阻塞队列类型
    1. 当workQueue已满，线程池线程数 > corePoolSize && 线程池线程数 < maximumPoolSize  时，新提交任务会创建新线程执行
    2. 当workQueue已满，线程池线程数 > maximumPoolSize，任务由RejectedExecutionHandler处理
* keepAliveTime：空闲线程多久后被回收
    1. 当线程池中线程数超过corePoolSize，超过corePoolSize这些线程的空闲时间达到keepAliveTime时，回收这些线程
    2. 当设置allowCoreThreadTimeOut(true)时，线程池中corePoolSize范围内的线程空闲时间达到keepAliveTime也将回收

## newFixedThreadPool
```java
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,0L, TimeUnit.MILLISECONDS,new LinkedBlockingQueue<Runnable>());
}
```
* Executors.newFixedThreadPool(2)：提交6个任务时，有两个任务可以运行，4个任务进入workQueue

## newCachedThreadPool
```java
public static ExecutorService newCachedThreadPool(ThreadFactory threadFactory) {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,60L, TimeUnit.SECONDS,new SynchronousQueue<Runnable>(),threadFactory);
}
```
* 当有新任务到来，则插入到SynchronousQueue中，由于SynchronousQueue是同步队列，因此会在池中寻找可用线程来执行，若有可以线程则执行，若没有可用线程则创建一个线程来执行该任务；若池中线程空闲时间超过指定大小，则该线程会被销毁

## 任务调度机制
1. 首先检测线程池运行状态，如果不是RUNNING，则直接拒绝，线程池要保证在RUNNING的状态下执行任务。
2. 如果workerCount < corePoolSize，则创建并启动一个线程来执行新提交的任务。
3. 如果workerCount >= corePoolSize，且线程池内的阻塞队列未满，则将任务添加到该阻塞队列中。
4. 如果workerCount >= corePoolSize && workerCount < maximumPoolSize，且线程池内的阻塞队列已满，则创建并启动一个线程来执行新提交的任务。
5. 如果workerCount >= maximumPoolSize，并且线程池内的阻塞队列已满, 则根据拒绝策略来处理该任务, 默认的处理方式是直接抛异常。

## 队列类型
* LinkedBlockingQueue
* ArrayBlockingQueue
* SynchronousQueue
* PriorityBlockingQueue

## 线程池监控
1. long getTaskCount()，获取已经执行或正在执行的任务数
2. long getCompletedTaskCount()，获取已经执行的任务数
3. int getLargestPoolSize()，获取线程池曾经创建过的最大线程数，根据这个参数，我们可以知道线程池是否满过
4. int getPoolSize()，获取线程池线程数
5. int getActiveCount()，获取活跃线程数（正在执行任务的线程数）
6. protected void beforeExecute(Thread t, Runnable r) // 任务执行前被调用
7. protected void afterExecute(Runnable r, Throwable t) // 任务执行后被调用
8. protected void terminated() // 线程池结束后被调用

## 异常处理
1. 使用Future记录异常信息
2. 重写ThreadPoolExecutor.afterExecute()，该方法在runWorker()中finally{}调用，默认是空实现
3. ThreadPoolExecutor.runWorker()中的异常会被jvm报告给UncaughtExceptionHandler，不适合submit()提交的任务，Callable中的异常会自己保存
4. 在线程池中手动处理异常
5. 没有任何异常处理的情况：ThreadPoolExecutor.runWorker()中进行了try...catch{}记录异常信息后抛出