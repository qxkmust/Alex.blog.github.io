# Spark的使用

### 安装

```
步骤一：
	在Spark官网http://spark.apache.org/downloads.html，这里使用spark-2.1.1-bin-hadoop2.7.tgz2.11.8.tar.gz
步骤二：
	使用sftp工具上传到VMWare主机hadoop02（192.168.8.105）的/opt目录并解压，重命名解压后的目录名称spark-2.1.1
步骤三：配置信息
	①进入/opt/spark-2.1.1,将当前目录配置到系统环境变量中：
        ①编辑/etc/profile文件，增加
                export SPARK_HOME=/opt/spark-2.1.1
                export PATH=$PATH:$SPARK_HOME/bin
		②使配置立刻生效,source /etc/profile
		③分发到hadoop03、hadoop04
			xsync /etc/profile
步骤四：验证
	①开启spark shell客户端，使用命令:bin/spark-shell
		jps查看SparkSubmit进程是否存在
	②官方案例一：蒙特卡罗方法推算圆周率
bin/spark-submit \
--class org.apache.spark.examples.SparkPi \
--executor-memory 1G \
--total-executor-cores 2 \
./examples/jars/spark-examples_2.11-2.1.1.jar \
100
	③登录hadoop02:4040查看程序运行的状态
```

### 基本架构

![Spark基本架构](D:\Git\MyRepository\Alex.blog.github.io\Images\Spark基本架构.webp)

```
Spark应用程序相关的几个术语：
	- Cluster Manager，Spark的集群管理器，主要负责对整个集群资源的分配和管理。根据部署模式的不同，可以分为三种模式：Local，Standalone，Yarn
    - Worker：集群中任何可以运行Application代码的节点，类似于YARN中的NodeManager节点。在Spark on Yarn模式中指的就是NodeManager节点；
    - Executor：Application运行在Worker 节点上的一个进程，该进程负责运行Task，并且负责将数据存在内存或者磁盘上，每个Application都有各自独立的一批Executor。
    - SparkContext：由用户程序启动，通过资源调度模块与Executor通信。
    - Driver：运行Application的main()函数，并创建SparkContext。其中创建SparkContext的目的是为了准备Spark应用程序的运行环境。在Spark中由SparkContext负责和ClusterManager通信，进行资源的申请、任务的分配和监控等；当Executor部分运行完毕后，Driver负责将SparkContext关闭。
```

### 基本概念

```
一个 Spark 应用程序包括 Job、Stage 以及 Task 三个概念：

　　①Job 是以 Action 方法为界，遇到一个 Action 方法则触发一个 Job；

　　②Stage 是 Job 的子集，以 RDD 宽依赖(即 Shuffle)为界，遇到 Shuffle 做一次划分；

　　③Task 是 Stage 的子集，以并行度(分区数)来衡量，分区数是多少，则有多少个 task。
```

### Spark对比Mapreduce

| 指标           | Spark                | Mapreduce       |
| -------------- | -------------------- | --------------- |
| 分布式并行计算 | 实时运算             | 离线运算        |
| 实时性         | 高                   | 低              |
| 容错性         | 高                   | 低·             |
| 操作算子       | 丰富的转换、执行算子 | 只有map、reduce |

### Spark高容错性

```
高容错性体现在两点：
	①spark支持DAG图的分布式并行计算（简单介绍以下spark DAG：即有向无环图，描述了任务间的先后依赖关系，spark中rdd经过若干次transform操作，由于transform操作是lazy的，因此，当rdd进行action操作时，rdd间的转换关系也会被提交上去，得到rdd内部的依赖关系，进而根据依赖，划分出不同的stage。），它引进rdd弹性分布式数据集的概念，它是分布在一组节点中的只读对象集合，如果数据集一部分数据丢失，则可以根据血统来对它们进行重建；
	②另外在RDD计算时可以通过checkpoint来实现容错
```

### Spark几种模式

| 模式             | 依赖进程       | 资源调度者      | 任务执行者                                                 |
| ---------------- | -------------- | --------------- | ---------------------------------------------------------- |
| Local（单机）    | 无             | SparkSubmit     | SparkSubmit                                                |
| Standalone(集群) | Master和Worker | Master          | CoarseGrainedExecutorBackend<br/>（属于Worker的进程）      |
| Yarn(集群)       | HDFS和Yarn     | ResourceManager | CoarseGrainedExecutorBackend<br/>（属于NodeManager的进程） |

##### Local模式运行流程

```
该模式就是在单机本地环境执行，主要用于程序测试。程序的所有部分，包括Client、Driver和Executor全部运行在客户端的SparkSubmit进程当中。
Local模式有三种启动方式：
	①local，只有一个Worker线程
	②local[K]，指定K个Worker线程数
	③local[*]，按照cpu核数尽可能多的设置Worker线程数
```

##### Standalone模式运行流程

```
根据提交方式不同又分为两种模式：
Standalone client模式：
    ①客户端使用命令spark-submit -–deploy-mode client提交任务后，会在客户端启动Driver进程。
    ②Driver 会向Master申请启动Application启动资源。
    ③资源申请成功后，Driver端会将task发送到worker端执行。
    ④worker端执行成功后将执行结果返回给Driver端
Standalone cluster模式：
	①客户端使用命令spark-submit -–deploy-mode cluster 后会启动spark-submit进程
    ②此进程为Driver向Master 申请资源。
    ③Master会随机在一台Worker节点来启动Driver进程。
    ④Driver启动成功后，spark-submit关闭，然后Driver向Master申请资源。
    ⑤Master接收到请求后，会在资源充足的Worker节点上启动Executor进程。
    ⑥Driver分发Task到Executor中执行。
```

##### Yarn模式运行流程

```
根据提交方式不同又分为两种模式：
```

![spark-yarn两种模式原理](D:\Git\MyRepository\Alex.blog.github.io\Images\spark-yarn两种模式原理.png)

## Spark 任务调度流程

​	当 Driver 起来后，Driver 则会根据用户程序逻辑准备任务，并根据 Executor 资源情况逐步分发任务。Spark 的任务调度总体来说分两路进行，一路是 Stage 级的调度，一路是 Task级的调度，总体调度流程如下图所示：

![spark任务调度流程](D:\Git\MyRepository\Alex.blog.github.io\Images\spark任务调度流程.png)

#### Stage 级的调度

​	Spark 的任务调度是从 DAG 切割开始，主要是由 DAGScheduler 来完成。当遇到一个 Action 操作后就会触发一个 Job 的计算，并交给 DAGScheduler 来提交，它会根据 RDD 的血缘关系构成的 DAG 进行切分，将一个 Job

划分为若干 Stages，具体划分策略是，由最终的 RDD 不断通过依赖回溯判断父依赖是否是宽依赖，即以 Shuffle 为界，划分 Stage，窄依赖的 RDD 之间被划分到同一个Stage 中。

#### Spark Task 级调度

　　Spark Task 的调度是由 TaskScheduler 来完成，由前文可知，DAGScheduler 将Stage 打 包 到 TaskSet 交 给 TaskScheduler， TaskScheduler 会 将 TaskSet 封装为TaskSetManager 加入到调度队列中。

------

### Spark Standalone模式

##### 简介

```
由一个Master多个Worker构成的Spark集群，称为Standalone模式
```

##### 配置

```
步骤一:基本配置
    ①重命名/opt/spark-2.1.1/conf/slaves.template为slaves，编辑Worker节点地址：
        hadoop02
        hadoop03
        hadoop04
    ②重命名/opt/spark-2.1.1/conf/spark-env.sh.template为spark-env.sh，编辑spark-env.sh：
        #配置Master所在主机地址
        SPARK_MASTER_HOST=hadoop02
        #配置Master服务端口
        SPARK_MASTER_PORT=7077
    ③配置/opt/spark-2.1.1/sbin/spark-config.sh，增加：
        JAVA_HOME=/opt/jdk1.8.0_144
步骤二：配置历史服务JobHistoryServer
	①重命名/opt/spark-2.1.1/conf/spark-defaults.conf.template为spark-defaults.conf
	②写日志的配置
		修改spark-defaults.conf,增加：
			#开启日志记录功能
            spark.eventLog.enabled true
            #配置日志存放目录
            spark.eventLog.dir hdfs://hadoop02:9000/spark/logs
     ③日志访问的配置
     	修改/opt/spark-2.1.1/conf/spark-env.sh,增加：
     		#日志访问配置
            export SPARK_HISTORY_OPTS="-Dspark.history.ui.port=18080
            -Dspark.history.retainedApplications=30
            -Dspark.history.fs.logDirectory=hdfs://hadoop02:9000/spark/logs"
		参数描述：
		###################################################################
		spark.eventLog.dir：Application在运行过程中所有的信息均记录在该属性指定的路径下
        spark.history.ui.port=18080  WEBUI访问的端口号为18080
        spark.history.fs.logDirectory=hdfs://hadoop102:9000/directory配置了该属性后，在start-history-server.sh时就无需再显式的指定路径，Spark History Server页面只展示该指定路径下的信息
		spark.history.retainedApplications=30指定保存Application历史记录的个数，如果超过这个值，旧的应用程序信息将被删除，这个是内存中的应用数，而不是页面上显示的应用数。
		###################################################################
步骤三：分发到hadoop03、hadoop04
	xsync /opt/spark-2.1.1
	xsync /usr/local/bin/spark-batch-start
	xsync /usr/local/bin/spark-batch-stop
步骤四：非必须，编写批量开关脚本
	编辑批量开启/关闭spark集群的shell脚本
	①在/usr/local/bin下添加脚本spark-batch-start,编辑：
		#!/bin/bash
        #群起spark集群
        ssh hadoop02 "source /etc/profile;$SPARK_HOME/sbin/start-all.sh"
	①添加脚本spark-batch-stop,编辑：
		#!/bin/bash
        #群起spark集群
        ssh hadoop02 "source /etc/profile;$SPARK_HOME/sbin/stopt-all.sh"
    ③授权
     	chmod 755 /usr/local/bin/spark-batch-start
     	chmod 755 /usr/local/bin/spark-batch-stop
步骤五：验证
	①群起spark集群：spark-batch-start，出现以下进程
		##########################################
		主机					进程
		##########################################
		hadoop02			Master/Worker
		hadoop03			Worker
		hadoop04			Worker
		##########################################
	同时登录http://hadoop02:8080，可查看Master-Worker节点的运行状态
	②启动历史服务器：sbin/start-history-server.sh，在hadoop02出现JobHistoryServer进程
	③官方案例一：蒙特卡罗方法推算圆周率
bin/spark-submit \
--class org.apache.spark.examples.SparkPi \
--master spark://hadoop02:7077 \
--executor-memory 1G \
--total-executor-cores 2 \
./examples/jars/spark-examples_2.11-2.1.1.jar \
100
	④登录http://hadoop02:18080，查看历史服务
```

### Spark高可用（只针对Standalone模式）

```
Spark Master节点存在单点故障，需要使用Zookeeper监控Master节点的状态，如果Master节点挂掉，重新从Standby Master节点选举出新的Master。

具体配置:
	①配置/opt/spark-2.1.1/conf/spark-env.sh,
        注释掉：
            #配置Master所在主机地址
            # SPARK_MASTER_HOST=hadoop02
            #配置Master服务端口
            # SPARK_MASTER_PORT=7077
        增加配置：
            #配置高可用
            export SPARK_DAEMON_JAVA_OPTS="-Dspark.deploy.recoveryMode=ZOOKEEPER
            -Dspark.deploy.zookeeper.url=hadoop02,hadoop03,hadoop04
            -Dspark.deploy.zookeeper.dir=/spark"
     ②分发配置
     	xsync /opt/spark-2.1.1/conf/spark-env.sh
     ③验证
     	①在hadoop02，启动spark集群，使用命令
     		spark-batch-start
     		发现，Zookeeper启动了hadoop02的Master进程（节点状态Alive），hadoop03和hadoop04的Master进程（节点状态Standby）也被启动
     	②手动杀掉hadoop02上的Master进程，分别登录hadoop03:8080和hadoop04:8080观察其中一个的状态切换成了Alive Master，说明高可用配置完成
     	③测试在高可用模式下，将job提交到spark集群，如果在任务执行过程中，Alive Master挂掉，任务会继续交给另外一个Alive Master运行
     	官方案例一：蒙特卡罗方法推算圆周率
bin/spark-submit \
--class org.apache.spark.examples.SparkPi \
--master spark://hadoop02:7077,spark://hadoop03:7077,spark://hadoop04:7077 \   //spark集群
--executor-memory 1G \
--total-executor-cores 2 \
./examples/jars/spark-examples_2.11-2.1.1.jar \
100
```

### Spark Yarn模式

##### 简介

```
Spark 客户端直接连接 Yarn，不需要额外构建 Spark 集群。
```

##### 配置

```
步骤一：
	①配置/opt/hadoop02-2.7.2/etc/hadoop/yarn-site.xml，添加：
        <!--是否启动一个线程检查每个任务正使用的物理内存量，如果任务超出分配值，则直接将其杀掉，默认是true --> 
        <property>
          <name>yarn.nodemanager.pmem-check-enabled</name>
          <value>false</value>
        </property> 
        <!--是否启动一个线程检查每个任务正使用的虚拟内存量，如果任务超出分配值，则直接将其杀掉，默认是true --> 
        <property>
          <name>yarn.nodemanager.vmem-check-enabled</name>
          <value>false</value>
        </property>
    ②分发yarn-site.xml
    	xsync /opt/hadoop02-2.7.2/etc/hadoop/yarn-site.xml
步骤二：配置spark-yarn模式
	配置/opt/spark-2.1.1/con/spark-env.sh,添加：
		YARN_CONF_DIR=/opt/hadoop-2.7.2/etc/hadoop
步骤三：非必须，实现在Yarn监控页面可以跳转到Spark监控页面
	为了实现Yarn的监控页面（hadoop03:8088）可以直接跳转到Spark监控页面（hadoop02:18080），在/opt/spark-2.1.1/conf/spark-defaults.conf中，增加配置：
	spark.yarn.historyServer.address=hadoop02:18080
	spark.history.ui.port=18080
步骤四：验证
	①启动hdfs集群，yarn集群
	②在主机hadoop02，使用命令：
bin/spark-submit \
--class org.apache.spark.examples.SparkPi \
--master yarn \
--deploy-mode client \
./examples/jars/spark-examples_2.11-2.1.1.jar \
100
```

### 配置LZO/Snappy压缩

```
spark配置lzo/snappy压缩的前提是hadoop支持lzo/snappy压缩

步骤一：hadoop配置lzo/snappy压缩功能
	hadoop默认支持GzipCodec、 DefaultCodec、BZip2Codec，需要自己编译Snappy和LZO的组件，并在/opt/hadoop-2.7.2/etc/hadoop/core-site.xml增加：

<!-- 增加lzo和Snappy压缩方式，默认只支持GzipCodec、 DefaultCodec、BZip2Codec-->
<property>
  <name>io.compression.codecs</name>
  <value>
    org.apache.hadoop.io.compress.GzipCodec,
    org.apache.hadoop.io.compress.DefaultCodec,
    org.apache.hadoop.io.compress.BZip2Codec,
    com.hadoop.compression.lzo.LzoCodec,
    com.hadoop.compression.lzo.LzopCodec,
    org.apache.hadoop.io.compress.SnappyCodec
  </value>
</property>
<!-- 指定lzop的类-->
<property>
  <name>io.compression.codec.lzo.class</name>
  <value>com.hadoop.compression.lzo.LzoCodec</value>
</property>
<!-- 指定Snappy的类-->
<property>
  <name>io.compression.codec.snappy.class</name>
  <value>com.hadoop.compression.lzo.LzopCodec</value>
</property>

重启hadoop集群，使用命令：hadoop checknative，查看hadoop是否已经支持lzo/snappy压缩

步骤二：配置spark支持lzo/snappy
	①在/opt/spark-2.1.1/conf/spark-env.sh,增加:
        #开启lzo压缩，前提是hadoop已经开启了lzo压缩功能
        # 指定lzo本地类库目录
        SPARK_LIBRARY_PATH=$SPARK_LIBRARY_PATH:/opt/hadoop-2.7.2/lib/native

        # 指定hadoop-lzo.jar路径
        SPARK_CLASSPATH=$SPARK_CLASSPATH:/opt/hadoop-2.7.2/share/hadoop/common/lib/hadoop-lzo-0.4.20.jar

```

###  Spark连接Hive

```
使用内置Hive：
	Spark有内嵌的Hive，可以直接使用，无需配置
使用外部Hive：
	①将/opt/hive-1.2.1/conf/hive-site.xml拷贝或软连接到/opt/spark-1.2.1/conf目录
	②将jdbc-connector-java-bin.jar驱动拷贝到/opt/spark-1.2.1/jars目录
	③使用spark-sql命令，开启客户端（类似hive客户端），可以直接用hql操作
```

### SparkStreaming原理

![SparkStreaming原理](D:\Git\MyRepository\Alex.blog.github.io\Images\SparkStreaming原理.png)

```
原理：
Spark Streaming会创建一个Streaming Context，它的底层是SparkContext，也就是说它会将任务提交给SparkContext来执行，这也很好的解释了DStream是一系列的RDD
①当启动Spark Streaming应用的时候，首先会在一个节点的Executor上启动一个Receiver接受者
②然后当从数据源写入数据的时候会被Receiver接收，接收到数据之后Receiver会将数据Split成很多个block，然后备份到各个节点（Replicate Blocks 容灾恢复）
③然后Receiver向StreamingContext进行块报告，说明数据在那几个节点的Executor上
④接着在一定间隔时间内StreamingContext会将数据处理为RDD并且交给SparkContext划分到各个节点进行并行计算。
```

### Spark调优

#### 解决数据倾斜问题

![spark解决数据倾斜总结](D:\Git\MyRepository\Alex.blog.github.io\Images\spark解决数据倾斜总结.png)