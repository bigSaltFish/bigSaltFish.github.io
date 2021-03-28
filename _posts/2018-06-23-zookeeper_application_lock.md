---
layout: default
title: zookeeper应用之分布式锁
#excerpt: 
---
&ensp;&ensp;前一段时间有讨论过用redis来实现分布式锁，讲到setNx不是原子性、redis新的set方法及其误删和守护线程，还为了原子性不得不使用redis的脚本。虽然最终分布式锁的这个效果是实现了，但是，不够优雅。这里讨论一下zookeeper对分布式锁的实现。  

&ensp;&ensp;首先说一下zk中节点的类型，共分为四个类型：持久节点、临时节点和持久有序节点、临时有序节点。  
&ensp;&ensp;什么是持久节点呢？字面意思，就是持久化的节点，当创建持久节点后，客户端断开了和服务器的链接，持久节点仍旧存在，与之相反的就是临时节点了，临时节点会在客户端断开链接后消失(客户端主动自刎或者zk服务器清理门户)。  
&ensp;&ensp;什么是顺序节点呢？先说下非顺序节点，比如节点“lock”，你只可以创建一次，创建第二次就会提示你“NodeExistsException”

    Exception in thread "main" org.apache.zookeeper.KeeperException$NodeExistsException: KeeperErrorCode = NodeExists for /lock   

&ensp;&ensp;假如使用的是顺序节点呢，上面这个异常就不会出现了，顺序节点会在节点名字后面追加一个序号，如果你使用“/lock”作为顺序节点，实际节点路径是“/lock0000000001”，十位数量的坑，绝对够用了。


&ensp;&ensp;再来说一下zk的数据结构.zk是树形的数据结构,   
![zk数据结构图]({{site.url}}/assets/2018-06-23-zookeeper_application_lock/zk_data_type.jpg)   
我们可以利用最左节点这个特性来实现分布式锁：如果有多个请求过来，会依次挂在指定节点下，我们每次取最左边的那个节点，执行完以后删除节点，让第二左节点接棒成为最左节点。
  
&ensp;&ensp;下面就是具体的代码了：
先看下Main.java，这个是入口   

	public class Main {
		private static final Logger LOG = LoggerFactory.getLogger(Main.class);
		private static final int SESSION_TIMEOUT = 10000;
		private static final String CONNECTION_STRING = "192.168.2.201:2181,192.168.2.201:2182,192.168.2.201:2183";
	
		static final String GROUP_PATH = "/lockParent";
		static final String SUB_PATH = GROUP_PATH + "/lock";
	
	
		public static void main(String[] args) {
			for (int i = 0; i < DistributedLock.THREAD_NUM; i++) {
				final int threadId = i + 1;
				new Thread() {
					@Override
					public void run() {
						try {
							DistributedLock dc = new DistributedLock(threadId, GROUP_PATH, SUB_PATH);
							// 1.实例化zk客户端
							dc.createConnection(CONNECTION_STRING, SESSION_TIMEOUT);
							// 2.创建公共的lock父节点(有/无视 无/创建)
							dc.createParent(GROUP_PATH, "该节点由线程" + threadId + "创建", true);
							// 3.创建子节点
							dc.createChilden();
							// 4.判断当前节点是不是罪左节点，是的话则成功获得锁
							if (dc.checkMinPath()) {
								dc.getLockSuccess();
							}
						} catch (Exception e) {
							LOG.error("【第" + threadId + "个线程】 抛出的异常：");
							e.printStackTrace();
						}
					}
				}.start();
			}
			
			try {
				DistributedLock.threadSemaphore.await();
				LOG.info("所有线程运行结束!");
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
	
		}
	}

还有具体的执行类 DistributedLock.java

	import org.slf4j.Logger;
	import org.slf4j.LoggerFactory;
	import org.apache.zookeeper.*;
	import org.apache.zookeeper.data.Stat;
	
	import java.util.List;
	import java.io.IOException;
	import java.util.Collections;
	import java.util.concurrent.CountDownLatch;
	
	public class DistributedLock implements Watcher {
		private static final Logger LOG = LoggerFactory.getLogger(DistributedLock.class);
		
		/** 父lock节点，持久有序节点，本demo所有的锁都是在这个节点下面 */
		private String groupPath;
		/** 具体的锁节点，隶属于groupPath下 */
		private String subPath;
		/** 当前线程信息，做日志输出时候用到 */
		private String LOG_PREFIX_OF_THREAD;
		
		/** zk的客户端连接 */
		private ZooKeeper zk = null;
		/** 本节点的path */
		private String selfPath;
		/** pre节点的path */
		private String waitPath;
		/** 模拟的请求个数 */
		static final int THREAD_NUM = 3;
		/** 确保连接zk成功才开始工作，避免抛出连接丢失异常 
		 * <span>org.apache.zookeeper.KeeperException$ConnectionLossException: KeeperErrorCode = ConnectionLoss for /lockParent</span>
		 */
		private CountDownLatch connectedSemaphore = new CountDownLatch(1);
		/** 确保所有线程运行结束； */
		static final CountDownLatch threadSemaphore = new CountDownLatch(THREAD_NUM);
	
		public DistributedLock(int threadId,String groupPath,String subPath) {
			this.groupPath = groupPath;
			this.subPath = subPath;
			LOG_PREFIX_OF_THREAD = "【第" + threadId + "个线程】";
		}
	
		/**
		 * 获取锁成功
		 */
		public void getLockSuccess() throws KeeperException, InterruptedException {
			if (zk.exists(this.selfPath, false) == null) {
				LOG.error(LOG_PREFIX_OF_THREAD + "本节点已不在了...");
				return;
			}
			LOG.info(LOG_PREFIX_OF_THREAD + "获取锁成功，赶紧干活！");
			Thread.sleep(5000);
			LOG.info(LOG_PREFIX_OF_THREAD + "删除本节点：" + selfPath);
			zk.delete(this.selfPath, -1);
			releaseConnection();
			threadSemaphore.countDown();
		}
	
		/**
		 * 检查自己是不是最小的节点
		 * @return
		 */
		public boolean checkMinPath() throws KeeperException, InterruptedException {
			List<String> subNodes = zk.getChildren(groupPath, false);
			Collections.sort(subNodes);
			int index = subNodes.indexOf(selfPath.substring(groupPath.length() + 1));
			switch (index) {
				case -1: {
					LOG.error(LOG_PREFIX_OF_THREAD + "父lock节点下找不到本节点:" + selfPath);
					return false;
				}
				case 0: {
					LOG.info(LOG_PREFIX_OF_THREAD + "本节点是最左节点:" + selfPath);
					return true;
				}
				default: {
					this.waitPath = groupPath + "/" + subNodes.get(index - 1);
					LOG.info(LOG_PREFIX_OF_THREAD + "获取子节点中，排在我前面的节点是:" + waitPath);
					try {
						zk.getData(waitPath, true, new Stat());
					} catch (KeeperException e) {
						if (zk.exists(waitPath, false) == null) {
							LOG.info(LOG_PREFIX_OF_THREAD + "子节点中，排在我前面的" + waitPath + "已失踪，幸福来得太突然?");
							return checkMinPath();
						} else {
							throw e;
						}
					}
					return false;
				}
			}
	
		}
	
		/**
		 * 创建ZK连接
		 * @param connectString    ZK服务器地址列表
		 * @param sessionTimeout   Session超时时间
		 */
		public void createConnection(String connectString, int sessionTimeout) throws IOException, InterruptedException {
			zk = new ZooKeeper(connectString, sessionTimeout, this);
			connectedSemaphore.await();
		}
		
		/**
		 * 创建子节点
		 * @return
		 */
		public void createChilden() throws KeeperException, InterruptedException {
			selfPath = zk.create(subPath, null, ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL_SEQUENTIAL);
			LOG.info(LOG_PREFIX_OF_THREAD + "创建锁路径:" + selfPath);
		}
	
		/**
		 * 创建节点
		 * @param path 节点path
		 * @param data 初始数据内容
		 * @return
		 */
		public void createParent(String path, String data, boolean needWatch)
				throws KeeperException, InterruptedException {
			if (null == zk.exists(path, needWatch)) {
				String createResult = this.zk.create(path, data.getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE,
						CreateMode.PERSISTENT);
				LOG.info(LOG_PREFIX_OF_THREAD + "节点创建成功, Path: " + createResult + ", content: " + data);
			}
		}
		
		/**
		 * 关闭ZK连接
		 */
		public void releaseConnection() {
			if (this.zk != null) {
				try {
					this.zk.close();
				} catch (InterruptedException e) {
				}
			}
			LOG.info(LOG_PREFIX_OF_THREAD + "释放连接");
		}
		
		public void process(WatchedEvent event) {
			if (event == null) {
				return;
			}
			Event.KeeperState keeperState = event.getState();
			Event.EventType eventType = event.getType();
			if (Event.KeeperState.SyncConnected == keeperState) {
				if (Event.EventType.None == eventType) {
					LOG.info(LOG_PREFIX_OF_THREAD + "成功连接上ZK服务器");
					connectedSemaphore.countDown();
				} else if (event.getType() == Event.EventType.NodeDeleted && event.getPath().equals(waitPath)) {
					LOG.info(LOG_PREFIX_OF_THREAD + "收到情报，排我前面的家伙已挂，我是不是可以出山了？");
					try {
						if (checkMinPath()) {
							getLockSuccess();
						}
					} catch (Exception e) {
						e.printStackTrace();
					}
				}
			} else if (Event.KeeperState.Disconnected == keeperState) {
				LOG.info(LOG_PREFIX_OF_THREAD + "与ZK服务器断开连接");
			} else if (Event.KeeperState.AuthFailed == keeperState) {
				LOG.info(LOG_PREFIX_OF_THREAD + "权限检查失败");
			} else if (Event.KeeperState.Expired == keeperState) {
				LOG.info(LOG_PREFIX_OF_THREAD + "会话失效");
			}
		}
	}   

运行Main类，可以清楚的看到三个线程先后获取锁、等待获取锁的信息   

	2018-06-23 23:54:56,656 - 【第3个线程】成功连接上ZK服务器
	2018-06-23 23:54:56,867 - 【第3个线程】创建锁路径:/lockParent/lock0000000007
	2018-06-23 23:54:56,923 - 【第3个线程】本节点是最左节点:/lockParent/lock0000000007
	2018-06-23 23:54:56,936 - 【第3个线程】获取锁成功，赶紧干活！
	2018-06-23 23:54:59,604 - 【第2个线程】成功连接上ZK服务器
	2018-06-23 23:54:59,612 - 【第1个线程】成功连接上ZK服务器
	2018-06-23 23:54:59,686 - 【第2个线程】创建锁路径:/lockParent/lock0000000008
	2018-06-23 23:54:59,698 - 【第1个线程】创建锁路径:/lockParent/lock0000000009
	2018-06-23 23:54:59,706 - 【第1个线程】获取子节点中，排在我前面的节点是:/lockParent/lock0000000008
	2018-06-23 23:54:59,709 - 【第2个线程】获取子节点中，排在我前面的节点是:/lockParent/lock0000000007
	2018-06-23 23:55:01,937 - 【第3个线程】删除本节点：/lockParent/lock0000000007
	2018-06-23 23:55:01,963 - 【第2个线程】收到情报，排我前面的家伙已挂，我是不是可以出山了？
	2018-06-23 23:55:01,979 - 【第2个线程】本节点是最左节点:/lockParent/lock0000000008
	2018-06-23 23:55:01,993 - 【第2个线程】获取锁成功，赶紧干活！
	2018-06-23 23:55:02,029 - 【第3个线程】释放连接
	2018-06-23 23:55:06,993 - 【第2个线程】删除本节点：/lockParent/lock0000000008
	2018-06-23 23:55:07,008 - 【第1个线程】收到情报，排我前面的家伙已挂，我是不是可以出山了？
	2018-06-23 23:55:07,032 - 【第1个线程】本节点是最左节点:/lockParent/lock0000000009
	2018-06-23 23:55:07,039 - 【第2个线程】释放连接
	2018-06-23 23:55:07,062 - 【第1个线程】获取锁成功，赶紧干活！
	2018-06-23 23:55:12,063 - 【第1个线程】删除本节点：/lockParent/lock0000000009
	2018-06-23 23:55:12,107 - 【第1个线程】释放连接
	2018-06-23 23:55:12,107 - 所有线程运行结束!


假如觉得上面的实现不够简洁，curator开源项目提供zookeeper分布式锁实现   
这是maven依赖

	<dependency>
		<groupId>org.apache.curator</groupId>
		<artifactId>curator-recipes</artifactId>
		<version>4.0.0</version>
	</dependency>   

具体代码：   

	public static void main(String[] args) throws Exception {
		//创建zookeeper的客户端
		RetryPolicy retryPolicy = new ExponentialBackoffRetry(1000, 3);
		CuratorFramework client = CuratorFrameworkFactory.newClient("10.21.41.181:2181,10.21.42.47:2181,10.21.49.252:2181", retryPolicy);
		client.start();
		//创建分布式锁, 锁空间的根节点路径为/curator/lock
		InterProcessMutex mutex = new InterProcessMutex(client, "/curator/lock");
		mutex.acquire();
		//获得了锁, 进行业务流程
		System.out.println("Enter mutex");
		//完成业务流程, 释放锁
		mutex.release();
		//关闭客户端
		client.close();
	}   

短短的十行代码就能实现锁的效果，只需要在acquire和release代码中间进行我们的业务操作即可，十分简洁明了