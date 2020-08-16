# Canal的使用

### 简介

```
canal是阿里开源的用于监控mysql数据变化，并将变化内容同步到其它组件（mysql/hbase/redis/oracle）
```

### 原理

```
类似mysql主从复制的过程：
    ①canal模拟mysql slave的交互协议，伪装自己为mysql slave，向mysql master发送dump协议
    ②mysql master收到dump请求，开始推送binary log给slave(也就是canal)
    ③canal解析binary log对象(原始为byte流)
```

### 架构

```
模块：
    server代表一个canal运行实例，对应于一个jvm
    instance对应于一个数据队列 （1个server对应1..n个instance)
其中，instance模块：
    eventParser (数据源接入，模拟slave协议和master进行交互，协议解析)
    eventSink (Parser和Store链接器，进行数据过滤，加工，分发的工作)
    eventStore (数据存储)
    metaManager (增量订阅&消费信息管理器)
```

### 安装

```
步骤一：
	从github官网https://github.com/alibaba/canal/releases下载相应版本的gz包，本机使用的canal.deployer-1.1.4.tar.gz
步骤二：
	使用sftp工具上传到VMWare主机hadoop02（192.168.8.105）的/opt目录并解压，重命名解压后的目录名称canal-1.1.4
步骤三：配置信息
	①进入/opt/canal-1.1.4,将当前目录配置到系统环境变量中：
        ①编辑/etc/profile文件，增加
                export CANAL_HOME=/opt/canal-1.1.4
                export PATH=$PATH:$CANAL_HOME/bin
		②使配置立刻生效,source /etc/profile
	②配置mysql开启bin-log日志记录
		编辑/etc/my.cnf,增加：
			#配置mysql主节点的唯一标识
				server-id = 1
            #必须，开启日志监控
    			log-bin=mysql-bin
			#必须，设置binlog格式为row(记录变化的数据到日志)
            	binlog_format=ROW
            #非必须，设置监控的数据库（生产中根据业务需求进行配置），不设置就是监控mysql服务中的所有数据库变化
            	binlog-do-db=数据库名
	②进入/opt/canal-1.1.4/conf/example，修改instance.properties，编辑：
		#canal节点的唯一标识，不能与mysql主机的server-id相同
			canal.instance.mysql.slaveId=0
		#指定mysql主节点的地址
			canal.instance.master.address=hadoop02:3306
		#可远程访问mysql的用户名和密码
        	canal.instance.dbUsername=alex
        	canal.instance.dbPassword=alex123
    ③进入/opt/canal-1.1.4/conf，修改canal.properties，编辑：
    	#canal服务端口（默认11111）
    		canal.port = 11111 
		#canal instance名称，一个canal server对应多个instance
			canal.destinations = hadoop02-instance
			注意：①canal server会为每个instance在conf和logs下生成同实例名的空目录，需要手动在conf下添加instance.properties
步骤四：启动canal服务
	/opt/canal-1.1.4/bin/startup.sh
步骤五：验证
	jps查看是否出现CanalLauncher进程
	或者查看/opt/canal-1.1.4/logs/canal/canal.log打印服务日志
	
	验证CanalLauncher是否同步mysql的写操作：
	①在主机hadoop02，登录mysql客户端，对main_db下的tb_stu表进行写操作（会在/var/lib/mysql下生成新的mysql-bin文件）
	②查看/opt/canal-1.1.4/logs/example/example.log是否记录mysql-bin更新的日志信息
```

### Canal应用场景

```
场景1：同步缓存redis/全文搜索ES
　　canal一个常见应用场景是同步缓存/全文搜索，当数据库变更后通过binlog进行缓存/ES的增量更新。当缓存/ES更新出现问题时，应该回退binlog到过去某个位置进行重新同步，并提供全量刷新缓存/ES的方法，如下图1所示。
　　
场景2：
	另一种常见应用场景是下发任务，当数据变更时需要通知其他依赖系统。其原理是任务系统监听数据库变更，然后将变更的数据写入MQ/kafka进行任务下发，比如商品数据变更后需要通知商品详情页、列表页、搜索页等相关系统。这种方式可以保证数据下发的精确性，通过MQ发送消息通知变更缓存是无法做到这一点的，而且业务系统中不会散落着各种下发MQ的代码，从而实现了下发归集，如下图2所示。
```

![canal应用场景](D:\Git\MyRepository\Alex.blog.github.io\Images\canal应用场景.png)

