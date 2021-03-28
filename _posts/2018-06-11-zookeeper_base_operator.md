---
layout: default
title: zookeeper 基本命令和java api
excerpt: 本文章介绍了zookeeper的基础shell操作和java API的操作
---
# 1.Zookeeper命令工具
启动zookeeper服务后，连接到zookeeper服务：

		zkServer -server localhost:2181
执行结果如下：

	Connecting to localhost:2181
	2018-06-10 03:52:52,540 [myid:] - INFO  [main:Environment@100] - Client environment:zookeeper.version=3.4.10-39d3a4f269333c922ed3db283be479f9deacaa0f, built on 03/23/2017 10:13 GMT
	2018-06-10 03:52:52,562 [myid:] - INFO  [main:Environment@100] - Client environment:host.name=192.168.15.201
	2018-06-10 03:52:52,563 [myid:] - INFO  [main:Environment@100] - Client environment:java.version=1.8.0_171
	2018-06-10 03:52:52,578 [myid:] - INFO  [main:Environment@100] - Client environment:java.vendor=Oracle Corporation
	2018-06-10 03:52:52,579 [myid:] - INFO  [main:Environment@100] - Client environment:java.home=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.171-8.b10.el6_9.x86_64/jre
	2018-06-10 03:52:52,579 [myid:] - INFO  [main:Environment@100] - Client environment:java.class.path=/usr/local/zookeeper/zookeeper-3.4.10_1/bin/../build/classes:/usr/local/zookeeper/zookeeper-3.4.10_1/bin/../build/lib/*.jar:/usr/local/zookeeper/zookeeper-3.4.10_1/bin/../lib/slf4j-log4j12-1.6.1.jar:/usr/local/zookeeper/zookeeper-3.4.10_1/bin/../lib/slf4j-api-1.6.1.jar:/usr/local/zookeeper/zookeeper-3.4.10_1/bin/../lib/netty-3.10.5.Final.jar:/usr/local/zookeeper/zookeeper-3.4.10_1/bin/../lib/log4j-1.2.16.jar:/usr/local/zookeeper/zookeeper-3.4.10_1/bin/../lib/jline-0.9.94.jar:/usr/local/zookeeper/zookeeper-3.4.10_1/bin/../zookeeper-3.4.10.jar:/usr/local/zookeeper/zookeeper-3.4.10_1/bin/../src/java/lib/*.jar:/usr/local/zookeeper/zookeeper-3.4.10_1/bin/../conf:.:/usr/lib/jvm/jre-1.8.0-openjdk-1.8.0.171-8.b10.el6_9.x86_64/lib/dt.jar:/usr/lib/jvm/jre-1.8.0-openjdk-1.8.0.171-8.b10.el6_9.x86_64/lib/tools.jar
	2018-06-10 03:52:52,579 [myid:] - INFO  [main:Environment@100] - Client environment:java.library.path=/usr/java/packages/lib/amd64:/usr/lib64:/lib64:/lib:/usr/lib
	2018-06-10 03:52:52,580 [myid:] - INFO  [main:Environment@100] - Client environment:java.io.tmpdir=/tmp
	2018-06-10 03:52:52,580 [myid:] - INFO  [main:Environment@100] - Client environment:java.compiler=<NA>
	2018-06-10 03:52:52,581 [myid:] - INFO  [main:Environment@100] - Client environment:os.name=Linux
	2018-06-10 03:52:52,581 [myid:] - INFO  [main:Environment@100] - Client environment:os.arch=amd64
	2018-06-10 03:52:52,582 [myid:] - INFO  [main:Environment@100] - Client environment:os.version=2.6.32-431.el6.x86_64
	2018-06-10 03:52:52,582 [myid:] - INFO  [main:Environment@100] - Client environment:user.name=root
	2018-06-10 03:52:52,582 [myid:] - INFO  [main:Environment@100] - Client environment:user.home=/root
	2018-06-10 03:52:52,583 [myid:] - INFO  [main:Environment@100] - Client environment:user.dir=/usr/local/zookeeper/zookeeper-3.4.10/bin
	2018-06-10 03:52:52,589 [myid:] - INFO  [main:ZooKeeper@438] - Initiating client connection, connectString=localhost:2181 sessionTimeout=30000 watcher=org.apache.zookeeper.ZooKeeperMain$MyWatcher@25f38edc
	Welcome to ZooKeeper!
	2018-06-10 03:52:52,716 [myid:] - INFO  [main-SendThread(localhost:2181):ClientCnxn$SendThread@1032] - Opening socket connection to server localhost/0:0:0:0:0:0:0:1:2181. Will not attempt to authenticate using SASL (unknown error)
	JLine support is enabled
	2018-06-10 03:52:53,237 [myid:] - INFO  [main-SendThread(localhost:2181):ClientCnxn$SendThread@876] - Socket connection established to localhost/0:0:0:0:0:0:0:1:2181, initiating session
	2018-06-10 03:52:53,507 [myid:] - INFO  [main-SendThread(localhost:2181):ClientCnxn$SendThread@1299] - Session establishment complete on server localhost/0:0:0:0:0:0:0:1:2181, sessionid = 0x163e94ed0530000, negotiated timeout = 30000
	
	WATCHER::
	
	WatchedEvent state:SyncConnected type:None path:null

控制台会输出一些配置信息和“welcome to Zookeeper”，查看zookeeper的相关操作，可以输入 help 命令

	[zk: localhost:2181(CONNECTED) 4] help
	ZooKeeper -server host:port cmd args
	stat path [watch]
	set path data [version]
	ls path [watch]
	delquota [-n|-b] path
	ls2 path [watch]
	setAcl path acl
	setquota -n|-b val path
	history 
	redo cmdno
	printwatches on|off
	delete path [version]
	sync path
	listquota path
	rmr path
	get path [watch]
	create [-s] [-e] path data acl
	addauth scheme auth
	quit 
	getAcl path
	close 
	connect host:port

演示下常见的zookeeper基本操作<br>
## 1.创建一个节点
  节点的名字是zoo，节点的相关信息是myData

	[zk: localhost:2181(CONNECTED) 4] create /zoo myData
	Created /zoo
再查看下现在zookeeper包含的节点

	[zk: localhost:2181(CONNECTED) 6] ls /
	[zoo, zookeeper]
这时，可以看到zoo节点已经被创建

## 2.获取指定节点的信息
	
	[zk: localhost:2181(CONNECTED) 7] get /zoo
	myData
	cZxid = 0x100000006
	ctime = Sun Jun 10 10:35:55 PDT 2018
	mZxid = 0x100000006
	mtime = Sun Jun 10 10:35:55 PDT 2018
	pZxid = 0x100000006
	cversion = 0
	dataVersion = 0
	aclVersion = 0
	ephemeralOwner = 0x0
	dataLength = 6
	numChildren = 0

## 3.修改指定节点的信息

	[zk: localhost:2181(CONNECTED) 8] set /zoo animais
	cZxid = 0x100000006
	ctime = Sun Jun 10 10:35:55 PDT 2018
	mZxid = 0x100000007
	mtime = Sun Jun 10 10:39:06 PDT 2018
	pZxid = 0x100000006
	cversion = 0
	dataVersion = 1
	aclVersion = 0
	ephemeralOwner = 0x0
	dataLength = 7
	numChildren = 0
再查看修改后内容
	
	[zk: localhost:2181(CONNECTED) 9] get /zoo        
	animais
	cZxid = 0x100000006
	ctime = Sun Jun 10 10:35:55 PDT 2018
	mZxid = 0x100000007
	mtime = Sun Jun 10 10:39:06 PDT 2018
	pZxid = 0x100000006
	cversion = 0
	dataVersion = 1
	aclVersion = 0
	ephemeralOwner = 0x0
	dataLength = 7
	numChildren = 0

## 4.删除节点

	[zk: localhost:2181(CONNECTED) 12] delete /zoo
	[zk: localhost:2181(CONNECTED) 13] ls /
	[zookeeper]

结验证，zoo节点已被删除

 
# 2.java API的使用
首先需要在maven添加zookeeper的jar包

		<dependency>
			<groupId>org.apache.zookeeper</groupId>
			<artifactId>zookeeper</artifactId>
			<version>3.4.10</version>
		</dependency>

具体代码如下：
	public class ZookeeperDemo {
		/** 服务地址，ip:port，多个地址用逗号隔开 */
		private static final String SERVER_ADDRESS = "192.168.2.201:218,192.168.2.201:2182,192.168.2.201:2183";
		/** 超时时间，毫秒为单位 */
		private static final Integer TIMEOUT_MS = 3000;
		/** 节点名 */
		private static final String NODE_ZOO = "/zoo/mammal";
		
		public static void main(String[] args) throws Exception {
			ZooKeeper zk = new ZooKeeper(SERVER_ADDRESS, TIMEOUT_MS, new DemoWatcher());
			waitUntilConnected(zk);
			// 判断节点是否存在
			Stat stat = zk.exists(NODE_ZOO, false);
			
			if (null == stat) {// 节点不存在
				createNode(zk, NODE_ZOO, "i'm mammal", ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
			} else {
				getData(zk, NODE_ZOO, stat);
				System.out.println(stat.getVersion());
	
				updateNode(zk, NODE_ZOO, "i'm mammal haha", stat.getVersion());
			}
		}
	
		/**
		 * 根据指定节点名字，创建节点
		 * @param zk               zk客户端
		 * @param node_zoo         节点名
		 * @param data             节点信息
		 * @param ACL_LIST         访问控制列表，这里使用了完全开放的ACL，允许任何客户端对znode进行读写
		 * @param createMode       创建znode的类型：有两种类型的znode：短暂的和持久的。
		 * @throws KeeperException
		 * @throws InterruptedException
		 */
		public static void createNode(ZooKeeper zk, String node_zoo, String data,List<ACL> ACL_LIST,CreateMode createMode)
				throws KeeperException, InterruptedException {
			String createResult = zk.create(node_zoo, data.getBytes(), ACL_LIST ,createMode);
			System.out.println(createResult);
		}
	
		/**
		 * 删除节点
		 * @param zk               zk客户端
		 * @param node_zoo         节点名
		 * @param version          指定的版本，-1/无视版本
		 * @throws KeeperException
		 * @throws InterruptedException
		 */
		public static void deleteNode(ZooKeeper zk, String node_zoo, Integer version)
				throws KeeperException, InterruptedException {
			zk.delete(node_zoo, version);
		}
	
		/**
		 * 更新节点
		 * @param zk               zk客户端
		 * @param node_zoo         节点名
		 * @param data             节点数据
		 * @param version          指定的版本，-1/无视版本
		 * @throws KeeperException
		 * @throws InterruptedException
		 */
		public static void updateNode(ZooKeeper zk, String node_zoo, String data, Integer version)
				throws KeeperException, InterruptedException {
			zk.setData(node_zoo, data.getBytes(), version);
		}
	
		/**
		 * 根据节点名获取节点信息
		 * @param zk               zk客户端
		 * @param node_zoo         节点名
		 * @param stat             节点状态
		 * @throws KeeperException
		 * @throws InterruptedException
		 */
		public static void getData(ZooKeeper zk, String node_zoo, Stat stat) throws KeeperException, InterruptedException {
			byte[] bs = zk.getData(node_zoo, false, stat);
			System.out.println(new String(bs));
		}
	
		/**
		 * 监听类，接收来自zookeeper的回调
		 * @author Administrator
		 *
		 */
		static class DemoWatcher implements Watcher {
			public void process(WatchedEvent event) {
				System.out.println("----------->");
				System.out.println("path:" + event.getPath());
				System.out.println("type:" + event.getType());
				System.out.println("stat:" + event.getState());
				System.out.println("<-----------");
			}
		}
	
		/**
		 * 防止java程序未连上zookeeper的服务器而往下执行，需等待zk的状态码
		 * @param zooKeeper
		 */
		public static void waitUntilConnected(ZooKeeper zooKeeper) {
			CountDownLatch connectedLatch = new CountDownLatch(1);
			Watcher watcher = new ConnectedWatcher(connectedLatch);
			zooKeeper.register(watcher);
			if (States.CONNECTING == zooKeeper.getState()) {
				try {
					connectedLatch.await();
				} catch (InterruptedException e) {
					throw new IllegalStateException(e);
				}
			}
		}
	
		static class ConnectedWatcher implements Watcher {
			private CountDownLatch connectedLatch;
	
			ConnectedWatcher(CountDownLatch connectedLatch) {
				this.connectedLatch = connectedLatch;
			}
	
			public void process(WatchedEvent event) {
				if (event.getState() == KeeperState.SyncConnected) {
					connectedLatch.countDown();
				}
			}
		}
	}

