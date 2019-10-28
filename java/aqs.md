

```java
static final class Node {
	/** 获取共享锁时被阻塞 */
	static final Node SHARED = new Node();
	/** 获取独占锁时被阻塞 */
	static final Node EXCLUSIVE = null;
    
	/** 线程取消锁争抢 */
	static final int CANCELLED =  1;
	/** 线程被唤醒 */
	static final int SIGNAL    = -1;
	/** 线程在条件队列里等待 */
	static final int CONDITION = -2;
	/** 线程释放共享资源时需要通知节点 */
	static final int PROPAGATE = -3;

	/** 取值0、1、-1、-2、-3 */
	volatile int waitStatus;

	/** 前驱节点 */
	volatile Node prev;

	/** 后继节点 */
	volatile Node next;

	/** 当前线程 */
	volatile Thread thread;

	/**  */
	Node nextWaiter;
｝
```

