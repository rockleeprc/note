
## 安装

### 目录结构

* bin：下面存放的是ActiveMQ的启动脚本
* conf：里面是配置文件，重点关注的是activemq.xml、jetty.xml、jetty-realm.properties。在登录ActiveMQ Web控制台需要用户名、密码信息；在JMS CLIENT和ActiveMQ进行何种协议的连接、端口是什么等这些信息都在上面的配置文件中可以体现。
* data：目录下是ActiveMQ进行消息持久化存放的地方，默认采用的是kahadb，当然我们可以采用leveldb，或者采用JDBC存储到MySQL，或者干脆不使用持久化机制。
* webapps：注意ActiveMQ自带Jetty提供Web管控台
* lib：ActiveMQ为我们提供了分功能的JAR包，当然也提供了activemq-all-5.14.4.jar

### 启动
	# 解压
	tar -zxvf apache-activemq-5.14.4-bin.tar.gz 
	# 启动
	bin/activemq start
	bin/activemq stop


### web控制台
	# 验证
	http://192.168.33.11:8161/
	u:admin
	p:admin

* Messages Enqueued：表示生产了多少条消息，记做P
* Messages Dequeued：表示消费了多少条消息，记做C
* Number Of Consumers：表示在该队列上还有多少消费者在等待接受消息
* Number Of Pending Messages：表示还有多少条消息没有被消费，实际上是表示消息的积压程度，就是P-C

## P2P模型
### 创建连接工厂
	// activemq.xml中配置指定的用户、密码才能访问ActiveMQ
	ConnectionFactory connectionFactory = new ActiveMQConnectionFactory(ActiveMQConnectionFactory.DEFAULT_USER,
			ActiveMQConnectionFactory.DEFAULT_PASSWORD, ActiveMQConnectionFactory.DEFAULT_BROKER_BIND_URL);

### 创建Connection

	Connection connection = null;
	//创建连接，默认是关闭状态
	connection= connectionFactory.createConnection();
	connection.start();

### 创建Session
	//创建session，单线程
	Session session = connection.createSession(Boolean.FALSE,Session.AUTO_ACKNOWLEDGE);

### 创建Destination
	//消息发送和接受的地点，要么queue，要么topic
	Destination destination = session.createQueue("mq-queue");

### 创建MessageProducer
	//创建MessageProducer，并设置持久化方式，默认持久
	MessageProducer messageProducer = session.createProducer(destination);
	messageProducer.setDeliveryMode(DeliveryMode.NON_PERSISTENT);

### 创建Message
	//jms规范定义5中消息类型StreamMessage、MapMessage、TextMessage、ObjectMessage、BytesMessage
	//定义messsage类型，并发送
	TextMessage message  = session.createTextMessage();
	message.setText("this is TextMessage");
	messageProducer.send(message);

### 签收模式
* AUTO_ACKNOWLEDGE：表示在消费者receive消息的时候自动的签收
* CLIENT_ACKNOWLEDGE：表示消费者receive消息后必须手动的调用acknowledge()方法进行签收
* DUPS_OK_ACKNOWLEDGE：签不签收无所谓了，只要消费者能够容忍重复的消息接受，当然这样会降低Session的开销

### 持久化信息到mysql
在activemq.xml的<broker>节点中增加MySQL信息