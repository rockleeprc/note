### AQS类结构

```java
public abstract class AbstractQueuedSynchronizer
    extends AbstractOwnableSynchronizer
    implements java.io.Serializable {
    
    static final class Node {
        /** Marker to indicate a node is waiting in shared mode */
        /** 标记线程获取共享资源时阻塞 */
        static final Node SHARED = new Node();
        /** Marker to indicate a node is waiting in exclusive mode */
        /** 标记线程获取独占资源时被阻塞 */
        static final Node EXCLUSIVE = null;

        /** waitStatus value to indicate thread has cancelled */
        static final int CANCELLED =  1;
        /** waitStatus value to indicate successor's thread needs unparking */
        static final int SIGNAL    = -1;
        /** waitStatus value to indicate thread is waiting on condition */
        static final int CONDITION = -2;
        /**
         * waitStatus value to indicate the next acquireShared should
         * unconditionally propagate
         */
        static final int PROPAGATE = -3;

        /**
         * Status field, taking on only the values:
         *   SIGNAL:     The successor of this node is (or will soon be)
         *               blocked (via park), so the current node must
         *               unpark its successor when it releases or
         *               cancels. To avoid races, acquire methods must
         *               first indicate they need a signal,
         *               then retry the atomic acquire, and then,
         *               on failure, block.
         *   CANCELLED:  This node is cancelled due to timeout or interrupt.
         *               Nodes never leave this state. In particular,
         *               a thread with cancelled node never again blocks.
         *   CONDITION:  This node is currently on a condition queue.
         *               It will not be used as a sync queue node
         *               until transferred, at which time the status
         *               will be set to 0. (Use of this value here has
         *               nothing to do with the other uses of the
         *               field, but simplifies mechanics.)
         *   PROPAGATE:  A releaseShared should be propagated to other
         *               nodes. This is set (for head node only) in
         *               doReleaseShared to ensure propagation
         *               continues, even if other operations have
         *               since intervened.
         *   0:          None of the above
         *
         * The values are arranged numerically to simplify use.
         * Non-negative values mean that a node doesn't need to
         * signal. So, most code doesn't need to check for particular
         * values, just for sign.
         *
         * The field is initialized to 0 for normal sync nodes, and
         * CONDITION for condition nodes.  It is modified using CAS
         * (or when possible, unconditional volatile writes).
         */
        /** 当前线程等待状态 */
        /** CANCELLED=1：线程被取消 */
        /** SIGNAL=-1：线程需要被唤醒 */
        /** CONDITION=-2：线程在条件队列里等待 */
        /** PROPAGATE=-3：释放共享资源是需要通知其它节点 */
        volatile int waitStatus;

        /**
         * Link to predecessor node that current node/thread relies on
         * for checking waitStatus. Assigned during enqueuing, and nulled
         * out (for sake of GC) only upon dequeuing.  Also, upon
         * cancellation of a predecessor, we short-circuit while
         * finding a non-cancelled one, which will always exist
         * because the head node is never cancelled: A node becomes
         * head only as a result of successful acquire. A
         * cancelled thread never succeeds in acquiring, and a thread only
         * cancels itself, not any other node.
         */
        /** 当前节点的前驱节点 */
        volatile Node prev;

        /**
         * Link to the successor node that the current node/thread
         * unparks upon release. Assigned during enqueuing, adjusted
         * when bypassing cancelled predecessors, and nulled out (for
         * sake of GC) when dequeued.  The enq operation does not
         * assign next field of a predecessor until after attachment,
         * so seeing a null next field does not necessarily mean that
         * node is at end of queue. However, if a next field appears
         * to be null, we can scan prev's from the tail to
         * double-check.  The next field of cancelled nodes is set to
         * point to the node itself instead of null, to make life
         * easier for isOnSyncQueue.
         */
        /** 当前节点的后驱节点 */
        volatile Node next;

        /**
         * The thread that enqueued this node.  Initialized on
         * construction and nulled out after use.
         */
        /** 进入aqs队列的线程 */       
        volatile Thread thread;

        /**
         * Link to next node waiting on condition, or the special
         * value SHARED.  Because condition queues are accessed only
         * when holding in exclusive mode, we just need a simple
         * linked queue to hold nodes while they are waiting on
         * conditions. They are then transferred to the queue to
         * re-acquire. And because conditions can only be exclusive,
         * we save a field by using special value to indicate shared
         * mode.
         */
        Node nextWaiter;

        /**
         * Returns true if node is waiting in shared mode.
         */
        final boolean isShared() {
            return nextWaiter == SHARED;
        }

        /**
         * Returns previous node, or throws NullPointerException if null.
         * Use when predecessor cannot be null.  The null check could
         * be elided, but is present to help the VM.
         *
         * @return the predecessor of this node
         */
        final Node predecessor() throws NullPointerException {
            Node p = prev;
            if (p == null)
                throw new NullPointerException();
            else
                return p;
        }

        Node() {    // Used to establish initial head or SHARED marker
        }

        Node(Thread thread, Node mode) {     // Used by addWaiter
            this.nextWaiter = mode;
            this.thread = thread;
        }

        Node(Thread thread, int waitStatus) { // Used by Condition
            this.waitStatus = waitStatus;
            this.thread = thread;
        }
    }
    /** 
    ReentrantLock：当前线程获取锁的可重入次数 
    ReentrantReadWriteLock：高16位表示读状态（获取读锁的次数），低16位表示写状态（可重入次数）
    semaphore：当前可用信号量次数
    CountDownlatch：计数器当前值
    */
    private volatile int state;
    
    /** 
    结合锁实现同步队列，可以直接访问AQS内的变量
    每个条件变量对应一个队列，用来存放条件变量的await方法后被阻塞的线程 
    */
    public class ConditionObject implements Condition, java.io.Serializable {
        private static final long serialVersionUID = 1173984872572414699L;
        /** First node of condition queue. */
        /** 条件队列的头 */
        private transient Node firstWaiter;
        /** Last node of condition queue. */
        /** 条件队列的尾 */
        private transient Node lastWaiter;
    }
}

```

### 核心方法

* 独占
  * public final void acquire(int arg)
  * public final void acquireInterruptibly(int arg)
  * public final boolean release(int arg)
* 共享
  * public final void acquireShared(int arg)
  * public final void acquireSharedInterruptibly(int arg)
  * public final boolean releaseShared(int arg)

### aqs队列

```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) // 子类重写方法 第一次尝试获取锁
        // Node.EXCLUSIVE = null
        // addWaiter(Node.EXCLUSIVE) 入队
        // acquireQueued() 尝试获取锁，失败后park入队的线程
        && acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}

private Node addWaiter(Node mode) {
    Node node = new Node(Thread.currentThread(), mode);
    // Try the fast path of enq; backup to full enq on failure
    // 尝试快速入队
    Node pred = tail;
    // 如果tail节点不为null，cas方式将node设置为tail
    if (pred != null) {
        node.prev = pred; // 设置node前驱节点
        if (compareAndSetTail(pred, node)) {// cas设置tail
            pred.next = node;// 设置pred后继节点
            return node;
        }
    }
    // pred/tail == null 链表为空 || cas(pred,node)竞争失败
    enq(node);// 入队
    return node;
}

private Node enq(final Node node) {
    for (;;) {
        Node t = tail; // 第一次循环head/tail=null，第二次循环作为哨兵节点
        if (t == null) { // 第一次循环tail为null，初始化队列
            if (compareAndSetHead(new Node())) // 设置head节点，头节点的thread永远等于null
                tail = head; // head -> 哨兵 <- tail
        } else { //tail节点不为空，执行入队操作
            node.prev = t; // 设置node的前驱节点为哨兵节点 哨兵 <- node.prev  
            if (compareAndSetTail(t, node)) { // 设置node节点为tail node <- tail
                t.next = node; // 设置哨兵节点的后继节点为node 哨兵.next -> node
                return t; // 返回哨兵
            }
        }
    }
}

final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            // 获取node前驱节点
            final Node p = node.predecessor();
            // node.prev==head时 第二次尝试获取锁
            if (p == head && tryAcquire(arg)) {
                setHead(node);// node设置为head，head.thread永远是null
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            // 第二次获取锁失败（可能在for()中自旋），挂起刚入队node节点线程
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}

private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
    int ws = pred.waitStatus; // 初始状态为0
    if (ws == Node.SIGNAL) // 可以阻塞，等待被唤醒
        /*
             * This node has already set status asking a release
             * to signal it, so it can safely park.
             */
        return true;
    if (ws > 0) { // 节点取消获取锁，递归删除放弃锁的节点 
        /*
             * Predecessor was cancelled. Skip over predecessors and
             * indicate retry.
             */
        do {
            node.prev = pred = pred.prev;
        } while (pred.waitStatus > 0);
        pred.next = node;
    } else { // waitStatus不等于Node.SIGNAL，节点没有取消获取锁
        /*
             * waitStatus must be 0 or PROPAGATE.  Indicate that we
             * need a signal, but don't park yet.  Caller will need to
             * retry to make sure it cannot acquire before parking.
             */
        // 尝试修改node.prev.waitStatus为Node.SIGNAL，自己被park后无法修改，自己不能获取自己的状态
        // t2 修改t2.prev的waitStatus为-1，t2.waitStatus=0
        // t3 修改t3.prev的waitStatus为-1，t3.waitStatus=0
        compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
    }
    return false;
}
```

### 条件变量

```java
public Condition newCondition() {
    return sync.newCondition();
}
final ConditionObject newCondition() {
    //AbstractQueuedSynchronizer中的内部类，每次调用newCondition都会new一个新的ConditionObject对象
    return new ConditionObject(); 
}
```

### condition队列



**一个锁对应一个AQS队列，对应多个ConditionObject，每个ConditionObject都有自己的队列**

