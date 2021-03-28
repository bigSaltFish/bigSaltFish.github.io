---
layout: default
title: zookeeper下载安装
#excerpt: 我是摘要
---
  文中使用的是zookeeper3.4.10，openjdk8，centos 6.5

# 1.安装目录、下载压缩包、解压缩

	mkdir -p /usr/local/zookeeper
	cd /usr/local/zookeeper
	wget https://mirrors.tuna.tsinghua.edu.cn/apache/zookeeper/zookeeper-3.4.10/zookeeper-3.4.10.tar.gz
	tar -zxvf zookeeper-3.4.10.tar.gz

# 2.添加环境变量
	vim /etc/profile
	#Set ZooKeeper Enviroment
	export ZOOKEEPER_HOME=/usr/local/zookeeper/zookeeper-3.4.10_1
	export PATH=$PATH:$ZOOKEEPER_HOME/bin:$ZOOKEEPER_HOME/conf
source /etc/profile (立即刷新可见)

# 3.修改配置文件
	cp zoo_sample.cfg zoo.cfg
	#服务器与客户端之间交互的基本时间单元（ms）
	tickTime=2000
	#配置保存数据文件夹
	dataDir=/usr/local/zookeeper/zookeeper-3.4.10_1/data
	#配置保存日志文件夹，当此配置不存在时默认路径与dataDir一致
	dataLogDir=/usr/local/zookeeper/zookeeper-3.4.10/logs
	#客户端访问zookeeper的端口号
	clientPort=2181
	
	server.1=localhost:2287:3387
	server.2=localhost:2288:3388
	server.3=localhost:2289:3389

# 4.配置myid
  有一个灰常关键的设置，在每个zk server配置文件的dataDir所对应的目录下，必须创建一个名为myid的文件，其中的内容必须与zoo.cfg中server.x 中的x相同，即：
/usr/local/zookeeper/zookeeper-3.4.10/data/myid 中的内容为1，对应server.1中的1

# 5.启动验证
	/usr/local/zookeeper/zookeeper-3.4.10_1/bin/zkServer.sh start /usr/local/zookeeper/zookeeper-3.4.10_1/conf/zoo.cfg
	/usr/local/zookeeper/zookeeper-3.4.10_2/bin/zkServer.sh start
	/usr/local/zookeeper/zookeeper-3.4.10_3/bin/zkServer.sh start
启用成功后，输入 jps 看下进程
	20351 ZooKeeperMain
	20791 QuorumPeerMain
	20822 QuorumPeerMain
	20865 QuorumPeerMain
应该至少能看到以上几个进程。

可以启动客户端测试下：
	bin/zkCli.sh -server localhost:2181