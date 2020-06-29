# Zookeeper的部署

#### 安装部署

```
在VMWare节点hadoop02（192.168.8.105）上安装，然后xsync分发到别的节点
步骤一：
	在zookeeper官网https://zookeeper.apache.org/releases.html，下载相应版本的gz文件，使用zookeeper-3.4.14.tar.gz
步骤二：
	上传到linux服务器的/opt目录，解压到当前目录
步骤三：配置信息
	①进入/opt/zookeeper-3.4.14/bin，将当前目录配置到系统环境变量中：
		①编辑/etc/profile文件，增加
			export ZOOKEEPER_HOME=/opt/zookeeper-3.4.14
			export PATH=$PATH:$ZOOKEEPER_HOME/bin
		②使配置立刻生效,source /etc/profile
	②zookeeper的服务默认读取的是/opt/zookeeper-3.4.14/conf/zoo.cfg文件，需要将/opt/zookeeper-3.4.14/conf下的zoo.sample.cfg更名为zoo.cfg
步骤四：测试单节点
	①开启zookeeper服务端，zkServer.sh start
		启动后出现QuorumPeerMain的进程，zkServer.sh stop关闭服务端
	②开启zookeeper客户端，zkCli.sh
		启动后出现ZooKeeperMain进程，使用quit命令关闭客户端连接
步骤五：分发到集群
	将整个zookeeper-3.4.14目录分发到hadoop03和hadoop04节点
	xsync /zookeeper-3.4.14
步骤六:配置Zookeeper服务器编号
	①zookeeper集群中为了区分所在服务器，需要为服务器增加编号，在/opt/zookeeper-3.4.14/zkData中增加myid文件，编辑内容：
        hadoop02的编号是2
        hadoop03的编号是3
        hadoop04的编号是4
    ②在zoo.cfg中配置服务器编号和服务端口号
    	server.2=hadoop02:2888:3888
    	server.3=hadoop03:2888:3888
    	server.4=hadoop04:2888:3888
    	其中，2888用于主从通信，3888用于集群Leader选举
    ③将myid和zoo.cfg分发到其它节点
步骤七：启动zookeeper集群
	集群中有三个zookeeper节点，启动zookeeper集群的条件是：过半的节点上zookeeper服务被开启
	①在hadoop02和hadoop03节点，启动zookeeper服务
	②查看节点zookeeper的状态
		①zookeeper是按照投票选举（RAFT算法）的方式选出所在节点的zookeeper服务作为Leader，如果得票过半就是Leader
            如果先启动hadoop02节点，hadoop02会先为自己投一票，但是还少一票
            然后启动hadoop03节点，由于集群还没有选举出Leader，hadoop03先为自己投一票，hadoop02也为hadoop03投一票，选出hadoop03作为Leader，集群启动成功
        ②zkServer.sh status，查看节点的状态，hadoop02是follower，hadoop03是leader
  注意：
  	zookeeper集群的选举要满足半数机制：集群中半数以上机器存活，集群可用，所以zookeeper适用于奇数个节点的环境。
		
```

