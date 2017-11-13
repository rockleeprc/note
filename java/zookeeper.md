
## 安装

### 修改配文件
在hdp01、hdp02、hdp03上配置zookeeper，zookeeper的运行需要JDK，所以需要先安装JDK，配置zookeeper的环境变量

	[root@hdp02 local]# tar -zxvf zookeeper-3.4.5.tar.gz
	[root@hdp02 local]# more /etc/profile  
	export ZOOKEEPER_HOME=/usr/local/zookeer-3.4.5  
	PATH=.:$HADOOP_HOME/bin:$ZOOKEEPER_HOME/bin:$JAVA_HOME/bin:$PATH  
	[root@hadoop1 local]# source /etc/profile

修改zookeeper的配置文件，主要是修改dataDir和配置服务器名称与主机名映射

	[root@hdp02 local]# pwd
	/usr/local/zookeeper-3.4.5/conf
	[root@hdp02 zkData]# mv zoo_sample.cfg zoo.cfg
	[root@hdp02 conf]# more zoo.cfg
	dataDir=/usr/local/zookeeper-3.4.5/data
	server.1=hadoop1:2888:3888
	server.2=hadoop2:2888:3888
	server.3=hadoop3:2888:3888

在dataDir目录中创建名称为myid的文件，在内配置主机id

	[root@hdp02 zkData]# cat myid
	2

把配置zookeeper目录和profile发给其它主机

	[root@hdp02 zookeeper-3.4.5]# scp /etc/profile hdp01:/etc/profile
	[root@hdp02 zookeeper-3.4.5]# scp /etc/profile hdp03:/etc/profile
	[root@hdp02 zookeeper-3.4.5]# scp -r /usr/local/zookeeper-3.4.5 hdp01:/usr/local/
	[root@hdp02 zookeeper-3.4.5]# scp -r /usr/local/zookeeper-3.4.5 hdp03:/usr/local/

启动和查看状态
	
	#在hdp01、hdp02、hdp03上分别启动
	[root@hdp02 bin]# zkServer.sh start

	[root@hdp02 bin]# zkServer.sh status
	JMX enabled by default
	Using config: /usr/local/zookeeper-3.4.5/bin/../conf/zoo.cfg
	Mode: follower

	[root@hadoop2 bin]# zkServer.sh status
	JMX enabled by default
	Using config: /usr/local/zookeeper-3.4.5/bin/../conf/zoo.cfg
	Mode: leader

	[root@hadoop3 bin]# zkServer.sh status
	JMX enabled by default
	Using config: /usr/local/zookeeper-3.4.5/bin/../conf/zoo.cfg
	Mode: follower

zookeeper的日志文件

	[root@hdp02 bin]#pwd
	 /usr/local/zookeeper/bin

通过shell脚本启动zookeeper集群

	#!/bin/bash
	for host in hdp01 hdp02 hdp03
	do 
	 echo "$host is starting ......"
	 ssh $host "source /etc/profile;/usr/local/zookeeper/bin/zkServer.sh start"
	done

### 配置文件参数详解

	1.tickTime：CS通信心跳时间
	Zookeeper 服务器之间或客户端与服务器之间维持心跳的时间间隔，也就是每个 tickTime 时间就会发送一个心跳。tickTime以毫秒为单位。
	tickTime=2000  

	2.initLimit：LF初始通信时限
	集群中的follower服务器(F)与leader服务器(L)之间初始连接时能容忍的最多心跳数（tickTime的数量）。
	initLimit=5  

	3.syncLimit：LF同步通信时限
	集群中的follower服务器与leader服务器之间请求和应答之间能容忍的最多心跳数（tickTime的数量）。
	syncLimit=2 

	4.dataDir：数据文件目录
	Zookeeper保存数据的目录，默认情况下，Zookeeper将写数据的日志文件也保存在这个目录里。
	dataDir=/home/michael/opt/zookeeper/data 

	5.clientPort：客户端连接端口
	客户端连接 Zookeeper 服务器的端口，Zookeeper 会监听这个端口，接受客户端的访问请求。
	clientPort=2181 
	
	6.服务器名称与地址：集群信息（服务器编号，服务器地址，LF通信端口，选举端口）
	这个配置项的书写格式比较特殊，规则如下：
	server.N=YYY:A:B

## ZK shell

	bin/zkCli.sh 
	create /test hello
	get /test 
	set /test word
	delete /test


## Zookeeper Java API

### Client端连接zookeeper
	private static final String HOSTS = "hdp01:2181,hdp02:2181,hdp03:2181";
	private static final int TIMEOUT = 10000;
	private static final CountDownLatch LATCH = new CountDownLatch(1);
	private static ZooKeeper zkClient = null;

	@Before
	public static void init() throws IOException, InterruptedException {
		zkClient = new ZooKeeper(HOSTS, TIMEOUT, new Watcher() {
			// 事件监听回调方法
			public void process(WatchedEvent event) {
				System.out.println("getType:" + event.getType());
				System.out.println("getPath:" + event.getPath());
				System.out.println("getState:" + event.getState());
				if (LATCH.getCount() > 0 && event.getState() == KeeperState.SyncConnected) {
					System.out.println("KeeperState is SyncConnected");
					LATCH.countDown();
				}
				// zkClient.getData("", true, null);//持续监听
			}
		});
		// latch.await()方法执行时，方法所在的线程会等待，当latch的count减为0时，才会唤醒等待的线程
		LATCH.await();
	}

	@After
	public void destroy() throws InterruptedException {
		zkClient.close();
		System.out.println("zkClient is closed");
	}

	@Test
	public void testCreate() throws KeeperException, InterruptedException {
		// 参数1：要创建的节点的路径
		// 参数2：节点的数据
		// 参数3：节点的权限
		// 参数4：节点的类型
		String create = zkClient.create("/eclipse", "helloZK".getBytes(), Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
		System.out.println(create);
	}

	@Test
	public void testCreateChildren() throws KeeperException, InterruptedException {
		// 注册Watcher
		zkClient.getChildren("/eclipse", true);
		zkClient.create("/eclipse/ide1", "ide1".getBytes(), Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
		zkClient.create("/eclipse/ide2", "ide2".getBytes(), Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
	}

	@Test
	public void testGetChildren() throws KeeperException, InterruptedException {
		List<String> children = zkClient.getChildren("/eclipse", true);
		for (String string : children) {
			System.out.println(string);
		}
	}

	@Test
	public void testGetData() throws KeeperException, InterruptedException, UnsupportedEncodingException {
		// watch==true为使用new ZooKeeper()构造方法中的Watcher
		byte[] bytes = zkClient.getData("/eclipse", true, null);
		System.out.println(new String(bytes, "UTF-8"));
	}

	@Test
	public void testDelete() throws InterruptedException, KeeperException {
		// -1 删除任何节点的版本数据
		zkClient.delete("/eclipse/ide", -1);
	}

	@Test
	public void testSetData() throws KeeperException, InterruptedException {
		// -1 任何节点的版本数据
		zkClient.setData("/eclipse/ide1", "i'm a ide".getBytes(), -1);
	}

	@Test
	public void testExist() throws KeeperException, InterruptedException {
		Stat stat = zkClient.exists("/none", true);
		// 节点不存在Stat对象为null
		System.out.println(stat);
	}

### 分布式共享锁

	public class DistributedClientLock {
		private static final String HOSTS = "hdp01:2181,hdp02:2181,hdp03:2181";
		private static final int TIMEOUT = 10000;
		// latch.await()方法执行时，方法所在的线程会等待，当latch的count减为0时，才会唤醒等待的线程
		// private static final CountDownLatch LATCH = new CountDownLatch(1);
		private static ZooKeeper zkClient = null;
		private static final String LOCKNODE = "lock";
		private static final String SUBNODE = "sub";
		private static final String separator = "/";
		private static String subPath;
	
		public void connectZookeeper() throws IOException, InterruptedException, KeeperException {
			zkClient = new ZooKeeper(HOSTS, TIMEOUT, new Watcher() {
	
				public void process(WatchedEvent event) {
					// 只处理/lock下子节点变化的事件
					if (event.getType() == EventType.NodeChildrenChanged && (separator + LOCKNODE).equals(event.getPath())) {
						try {
							// 获取/lock下的子节点，并且监听
							List<String> children = zkClient.getChildren(separator + LOCKNODE, true);
							// 获取子节点名称
							String subNode = subPath.substring((separator + LOCKNODE + separator).length());
							// 索引自己的节点名称是否为最小
							Collections.sort(children);
							if (children.indexOf(subNode) == 0) {
								doSomething();
								// 执行业务逻辑后，重新注册一把锁，等待下次执行
								subPath = registLock();
							}
						} catch (KeeperException e) {
							e.printStackTrace();
						} catch (InterruptedException e) {
							e.printStackTrace();
						}
					}
				}
			});
			// 建立连接后先注册一把锁
			subPath = registLock();
			// 休眠一下
			Thread.sleep(new Random().nextInt(1000));
			// 从zk的/lock目录下，获取所有子节点，并且注册对父节点的监听
			List<String> children = zkClient.getChildren(separator + LOCKNODE, true);
			// 如果争抢资源的程序就只有自己，则可以直接去访问共享资源
			if (children.size() == 1) {
				doSomething();
				subPath = registLock();
			}
		}
	
		/**
		 * 注册锁到zk上
		 * 
		 * @return
		 * @throws KeeperException
		 * @throws InterruptedException
		 */
		private String registLock() throws KeeperException, InterruptedException {
			String path = zkClient.create(separator + LOCKNODE + separator + SUBNODE, null, Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL_SEQUENTIAL);
			return path;
		}
	
		/**
		 * 业务逻辑
		 * 
		 * @throws InterruptedException
		 * @throws KeeperException
		 */
		private void doSomething() throws InterruptedException, KeeperException {
			try {
				System.out.println("gain lock: " + subPath);
				Thread.sleep(2000);
			} finally {
				System.out.println("finished: " + subPath);
				// 访问完毕后，需要手动去删除之前的锁节点
				zkClient.delete(subPath, -1);
			}
		}
	
		public static void main(String[] args) throws IOException, InterruptedException, KeeperException {
			DistributedClientLock dcLock = new DistributedClientLock();
			dcLock.connectZookeeper();
			Thread.sleep(Long.MAX_VALUE);
		}
	}