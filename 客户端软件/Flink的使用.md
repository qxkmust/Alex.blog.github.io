# Flink的使用

### 安装

```
在主机hadoop02上安装：
 步骤一：
	从官网 https://flink.apache.org/downloads.html 下载相应版本的gz包，本机使用的flink-1.7.2-bin-scala_2.11.tgz
步骤二：
	使用sftp工具上传到VMWare主机hadoop02（192.168.8.105）的/opt目录并解压，重命名解压后的目录名称flink-1.7.2
步骤三：配置信息
	①进入/opt/flink-1.7.2,将当前目录配置到系统环境变量中：
        ①编辑/etc/profile文件，增加
                export FLINK_HOME=/opt/flink-1.7.2
                export PATH=$PATH:$FLINK_HOME/bin
		②使配置立刻生效,source /etc/profile
		③分发 /etc/profile
	②运行模式配置，参考Standalone模式、或者yarn模式
步骤四：分发到hadoop03、hadoop04
	xsync /opt/flink-1.7.2
步骤五：启动
	/opt/flink-1.7.2/bin/start-cluster.sh
步骤六：验证
	①登录192.168.8.105:8081,访问flink监控界面
	②jps查看，在hadoop02下出现StandaloneSessionClusterEntrypoint进程（Master），hadoop03和hadoop04下出现TaskManagerRunner进程（Slave）
```

### 配置Standalone模式（常用于测试）

```
配置Standalone模式，就是配置flink一主（Master）多从（Slave）模式
步骤一：配置信息
    ②修改/opt/flink-1.7.2/conf/flink-conf.yaml
        #指定jobmanager服务地址
        	jobmanager.rpc.address:(这里有一个空格)hadoop02
    ③指定从机taskmanager，/opt/flink-1.7.2/conf/slave,
            hadoop03
            hadoop04
    ③分发配置文件
步骤二：启动
	/opt/flink-1.7.2/bin/start-cluster.sh
步骤三：验证
	方式一：Flink监控界面上传任务jar包
        ①将idea下big-data工程big-data-11-flink项目打成（带依赖jar包）jar包big-data-11-flink-1.0-SNAPSHOT.jar，注意这个jar包是由maven-shade-plugin生成，没有主函数，需要手动在jar包中/META-INF/MANIFEST.MF文件中添加：Main-Class: flink_streaming.WordCount
        ②访问http://hadoop02:8081监控界面，在菜单“Submit new Job" -> 手动添加要运行的任务jar包 -> Submit运行任务
    方式二：flink命令运行任务jar包
    	①将big-data-11-flink-1.0-SNAPSHOT.jar上传到hadoop02（StandaloneSessionClusterEntrypoint进程所在Master主机）/opt/flink-1.7.2/mydata/下
    	②使用命令：
    		/opt/flink-1.7.2/bin/flink run -c flink_streaming.WordCount /opt/flink-1.7.2/mydata/big-data-11-flink-1.0-SNAPSHOT.jar
    		【命令说明】 flink run -c【指定主类】 -p【任务并行度】 【任务jar包的路径】 多个输入参数（空格隔开）
```

### 配置Yarn模式（常用于生产）

```
配置Yarn模式，就是将Flink任务交给Yarn进行资源调度，需要Flink支持hadoop

步骤一：配置Flink支持hadoop的jar包
	默认的flink版本是不支持hadoop，需要从maven仓库下载对应flink版本的jar包，并上传到/opt/flink-1.7.2/lib下，我们使用flink-shaded-hadoop2-uber-1.7.2.jar
步骤二：启动
	①启动yarn-session
	/opt/flink-1.7.2/bin/yarn-session.sh -n 2 -s 2 -jm 200 -tm 100 -nm test -d
	【命令说明】yarn-session.sh -n【TaskManager数量】 -s【每个TaskManager的slot数量】 -jm【JobManager的内存】 -tm【每个taskManager的内存】 -nm【提交到yarn的任务名称】 -d【后台运行】
	②执行任务
		/opt/flink-1.7.2/bin/flink run -m yarn-cluster -c flink_streaming.WordCount /opt/flink-1.7.2/mydata/big-data-11-flink-1.0-SNAPSHOT.jar
		【命令说明】flink run -m yarn-cluster【指定jobmanager运行在yarn模式】 -c【指定主类】 -p【任务并行度】 【任务jar包的路径】 多个输入参数（空格隔开）
	
步骤三：验证
	访问yarn的web监控页面地址：hadoop03:8088，查看flink任务的执行日志
```

### Flink原理

```
四个组件：
	JobManager（作业管理器）
	ResourceManager（资源管理器）
	TaskManager（任务管理器）
	Dispatcher（分发器）
```

##### Standalone模式任务提交流程

![Standalone模式运行原理](D:\Git\MyRepository\Alex.blog.github.io\Images\Standalone模式运行原理.png)

##### Yarn模式任务提交流程

![Yarn模式运行原理](D:\Git\MyRepository\Alex.blog.github.io\Images\Yarn模式运行原理.png)

```
Flink 任务提交后，Client 向 HDFS 上传 Flink 的 Jar 包和配置，之后向 Yarn ResourceManager 提交任务，ResourceManager 分配 Container 资源并通知对应的NodeManager 启动 ApplicationMaster，ApplicationMaster 启动后加载 Flink 的 Jar 包和配置构建环境，然后启动 JobManager，之后 ApplicationMaster 向 ResourceManager申请资源启动 TaskManager ， ResourceManager 分 配 Container 资 源 后 ， 由ApplicationMaster 通 知 资 源 所 在 节 点 的 NodeManager 启 动 TaskManager ，NodeManager 加载 Flink 的 Jar 包和配置构建环境并启动 TaskManager，TaskManager
启动后向 JobManager 发送心跳包，并等待 JobManager 向其分配任务。
```

### flink操作hdfs

```
flink本身不依赖hdfs，如果需要连接hdfs进行读写操作，需要配置：
配置flink集群：
	①下载flink-shaded-hadoop2-uber-1.7.2.jar，放到/opt/flink-1.7.2/lib目录下
	②分发到hadoop03、hadoop04

客户端进行idea开发：
	①引入依赖：
        <dependency>
            <!--flink支持hadoop依赖-->
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-hadoop-compatibility_2.11</artifactId>
        </dependency>
        <dependency>
            <!--注入hadoop客户端依赖-->
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-client</artifactId>
            <version>2.7.2</version>
        </dependency>
    ②从hadoop集群上拷贝core-site.xml和hdfs-site.xml文件到项目的sources目录
    ③关闭权限检查，修改hdfs-site.xml
    	<!--关闭HDFS权限检查，不局限于root用户-->
        <property>
            <name>dfs.permissions.enable</name>
            <value>false</value>
        </property>
```

