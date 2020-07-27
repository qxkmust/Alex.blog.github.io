# kafka的使用

### 安装

```
步骤一：
	从官网http://kafka.apache.org/downloads下载相应版本的gz包，本机使用的kafka_2.11-0.11.0.0.tgz
步骤二：
	使用sftp工具上传到VMWare主机hadoop02（192.168.8.105）的/opt目录并解压，重命名解压后的目录名称kafka_2.11-0.11.0.0
步骤三：配置信息
	①进入/opt/kafka_2.11-0.11.0.0,将当前目录配置到系统环境变量中：
        ①编辑/etc/profile文件，增加
                export KAFKA_HOME=/opt/kafka_2.11-0.11.0.0
                export PATH=$PATH:$KAFKA_HOME/bin
		②使配置立刻生效,source /etc/profile
	②创建/logs目录
	③创建/data目录
	④编辑/opt/kafka_2.11-0.11.0.0/conf/server.properties，做以下修改
		#broker的全局唯一编号，不能重复
		broker.id=0
		#删除topic功能使能
		delete.topic.enable=true
		#运行日志存放路径
		log.dirs=/opt/kafka_2.11-0.11.0.0/data
		#配置zookeeper集群地址
		zookeeper.connect=hadoop02:2181,hadoop03:2181,hadoop04:2181
	⑤将/opt/kafka_2.11-0.11.0.0分发到主机hadoop03和hadoop04
		xsync /opt/kafka_2.11-0.11.0.0
步骤四：启动kafka服务
	使用命令kafka-server-start.sh -daemon ../config/server.properties
	或者在/usr/local/bin下新建shell文件kafka-batch-start，编辑：
```

```
#!/bin/bash
        #批量开启kafka服务
        for i in 2 3 4
        do
        echo -e ""
        echo -e "starting hadoop0$i kafka server"
        ssh hadoop0$i "source /etc/profile;$KAFKA_HOME/bin/kafka-server-start.sh -daemon $KAFKA_HOME/config/server.properties"
        done
```

```
给kafka-batch-start授权chmon 777 kafka-batch-start
```

### kafka架构

![kafka架构](D:\Git\MyRepository\Alex.blog.github.io\Images\kafka架构.png)

​			**10）**同一个组中的消费者数不大于分区数，因为超出的消费者没有分区对应，也就没有收不到消息offset

​			11) 0.8 以前，消费进度是直接写到 zookeeper 的，consumer 必须知道 zookeeper 的地址。这个方案有性能问题，0.9 的时候整体大改了一次，brokers 接管了消费进度，consumer 不再需要和 zookeeper 通信了。 

### kafka命令行操作

**操作主题topic的语法:** 

| 查看当前服务器中所有topic | kafka-topics.sh --list --zookeeper hadoop02:2181             |
| ------------------------- | ------------------------------------------------------------ |
| 创建topic                 | kafka-topics.sh --create --zookeeper hadoop02:2181 --topic mytopic --partitions 2 --replication-factor 2<br/>语法: kafka-topics.sh --create --zookeeper 【zk集群地址】 --topic 【主题名】 --partition 【分区数】 --replication-factor 【副本数】 |
| 查看某个topic详情         | kafka-topics.sh --describe --zookeeper hadoop02:2181 --topic mytopic<br/>语法：kafka-topics.sh --describe --zookeeper 【zk集群地址】 --topic 【主题名】 |
| 删除topic                 | kafka-topics.sh --delete --zookeeper hadoop02:2181 --topic mytopic<br/>语法：kafka-topics.sh --delete --zookeeper 【zk集群地址】 --topic 【要删除的主题名】 |

**操作主题topic的语法:** 

| 发送消息           | kafka-console-producer.sh --broker-list hadoop02:9092 --topic mytopic<br/>语法：kafka-console-producer.sh --broker-list 【当前kafka集群地址】 --topic 【向哪个主题发】 |
| ------------------ | ------------------------------------------------------------ |
| 消费消息           | kafka-console-consumer.sh --bootstrap-server hadoop02:9092 --topic mytopic --from-beginning<br/>语法：kafka-console-consumer.sh --bootstrap-server 【当前kafka集群地址】 --topic 【从哪个主题获取消息】 --from-beginning，表示接受所有的消息（包括离线和实时的消息） |
| 以消费者组消费消息 | 步骤一：在主机hadoop02、hadoop03中，配置/opt/kafka_2.11-0.11.0.0/config/consumer.properties中修改group.id=mygroup_alex<br/>步骤二：分别在三台主机开启消费者<br/>kafka-console-consumer.sh --bootstrap-server hadoop02:9092 --topic mytopic --consumer.config consumer.properties<br/>验证：在主机hadoop04上开启消息生产者，发现hadoop02、hadoop03交替获得消息<br/> |

### Kafka的生产者的消息发送确认机制

```
Ack=0，相当于异步发送，消息发送完毕即 offset 增加1，继续生产。
默认Ack=1，leader 收到 leader replica 对一个消息的接受 ack 才增加 offset，然后继续生产。
Ack=-1，leader 收到所有 replica 对一个消息的接受 ack 才增加 offset，然后继续生产。

注意，ack的默认值就是1。这个默认值其实就是吞吐量与可靠性的一个折中方案。生产上我们可以根据实际情况进行调整，比如如果你要追求高吞吐量，那么就要放弃可靠性。
```

| ack  | 消息一次性语义 | 是否会丢数据                                                 | 是否会重复                                                   |
| ---- | -------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 0    | At Mostly Once | 会<br>比如：broker收到消息之前就已经挂掉                     | 不会                                                         |
| 1    | At Mostly Once | 会<br>比如：<br>①broker中的leader收到消息，follower还<br>没有从leader同步消息就挂掉了，造成follower<br>数据丢失<br>②broker中的leader收到消息，follower还<br/>没有从leader同步消息，leader挂掉了，<br>也会造成follower数据丢失 | 不会                                                         |
| -1   | At Least Once  | 不会                                                         | 会<br>比如:<br>broker中的leader 和follower都同步完成，<br>此时leader挂了，生产者没有收到ack=-1的回复，会重复发送消息，造成数据重复 |



### kafka精确一次性

| Exactly Once | 存在问题                                                     | 解决办法                                                     |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 生产者端     | ①ack=-1无法解决数据重复<br>②幂等性无法解决跨分区跨会话数据重复 | ack=-1 和 幂等性 和 事务共同保证                             |
| 消费者端     | 在消费后，offset没有提交<br>①消费后，程序挂掉，offset没来及提交<br>②消费速度很慢，offset提交之前发生会话超时，<br>导致重分区，重分区后，消息被重复消费 | ①在程序中手动提交offset<br>②增加消费能力<br>③增大session.timeout.ms的大小（0.10版本默认30秒，0.11版本默认10秒），以免kafka还没有消费完就被关闭会话<br>④在下游对数据去重 |



### kafka分区分配策略

##### 一：消费者分区策略

如果consumer group 包含多个consumer，同一个组内同一时刻只会有一个消费者得到消息，这就涉及到partition的分配问题。一般有两种分区分配策略，***默认Range分配***：

①Range

 先把主题里的分区按照序号排序，然后把消费者按照字母排序，把【分区号】 % 【消费者总数】得到每个消费者消费的分区数。 

| 分区号                                                       | 消费者组编号                                           | 分配结果                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------ | ------------------------------------------------------------ |
| partition0	<br/>partition1	<br/>partition2	<br/>partition3	<br/>partition4	<br/>partition5	<br/>partition6	<br/> | consumer0	<br/>consumer1	<br/>consumer2	<br/> | consumer0：partition0、partition3、partition6<br/>consumer1：partition1、partition4<br/>consumer2：partition2、partition5<br/> |

②Round-Robin 循环分配

 把主题和分区组成topicAndPartition列表，再把列表按照hashcode排序，轮询分配给消费者。 

| 主题   | 分区号                                                       | 消费者组编号                                           | 假设按照<br/>【主题名】【分区号】<br/>的hash值排序如下       | 分配结果                                                     |
| ------ | ------------------------------------------------------------ | ------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| T1、T2 | partition0	<br/>partition1	<br/>partition2	<br/>partition3	<br/> | consumer0	<br/>consumer1	<br/>consumer2	<br/> | T10<br/>T20<br/>T11<br/>T21<br/>T12<br/>T22<br/>T13<br/>T23<br/> | consumer0：T10、T21、T13	<br/>consumer1：T20、T12、T23	<br/>consumer2：T11、T22	<br/> |

##### 二：生产者分区策略

生产中一般将producer发送的数据封装成一个ProducerRecord对象：

①ProducerRecord(String topic,Integer partition,String key,String value)

​	**partition号 = 指定的分区号**

②ProducerRecord(String topic,String key,String value)

​	**partition号 = 【key的hashcode值】% 【topic的分区数】**

注意： 这个hash值是通过kafka自己的hash算法计算的，不同于JAVA String.hashCode()

③ProducerRecord(String topic,String value)

​	**partition号 =**  **【随机生成一个自增的整数】% 【topic的分区数】**

### kafka监控

##### 简介：

```
支持管理多个Kafka集群、管理Kafka主题（包含查看、删除、创建等）、消费者组合消费者实例监控、消息阻塞告警、Kafka集群健康状态查看等 
```

##### 安装配置：

```
在hadoop02主机上安装
步骤一：
	在官网http://download.kafka-eagle.org/，下载版本kafka-eagle-web-1.3.7-bin.tar.gz
步骤二：
	上传到linux服务器/opt目录，解压到当前目录，重命名kafka-eagle-web-1.3.7
步骤三：配置信息
	①修改/opt/kafka_2.11-0.11.0.0/bin/kafka-server-start.sh
		①将if [ "x$KAFKA_HEAP_OPTS" = "x" ]; then
             export KAFKA_HEAP_OPTS="-Xmx1G -Xms1G"
            fi
        为
          if [ "x$KAFKA_HEAP_OPTS" = "x" ]; then
             export KAFKA_HEAP_OPTS="-server -Xms100m -Xmx200m -XX:PermSize=128m 
             -XX:+UseG1GC -XX:MaxGCPauseMillis=200 -XX:ParallelGCThreads=8
             -XX:ConcGCThreads=5 -XX:InitiatingHeapOccupancyPercent=70"
             export JMX_PORT="9999"
          fi
         ②分发配置
         	xsync /opt/kafka_2.11-0.11.0.0/bin/kafka-server-start.sh
     ②进入/opt/kafka-eagle-web-1.3.7，将当前目录配置到系统环境变量中：
		①编辑/etc/profile文件，增加
			export KE_HOME=/opt/kafka-eagle-web-1.3.7
			export PATH=$PATH:$KE_HOME/bin
		②使配置立刻生效,source /etc/profile
	 ③修改/opt/kafka-eagle-web-1.3.7/conf/system-config.properties
            ######################################
            # multi zookeeper&kafka cluster list
            ######################################
            kafka.eagle.zk.cluster.alias=cluster1
            cluster1.zk.list=hadoop02:2181,hadoop03:2181,hadoop04:2181
            ######################################
            # kafka offset storage
            ######################################
            cluster1.kafka.eagle.offset.storage=kafka
            ######################################
            # enable kafka metrics
            ######################################
            kafka.eagle.metrics.charts=true
            kafka.eagle.sql.fix.error=false
            ######################################
            # kafka jdbc driver address
            ######################################
            kafka.eagle.driver=com.mysql.jdbc.Driver
            kafka.eagle.url=jdbc:mysql://hadoop02:3306/ke?useUnicode=true&ch
            aracterEncoding=UTF-8&zeroDateTimeBehavior=convertToNull
            #以下为mysql远程账户和密码
            kafka.eagle.username=alex
            kafka.eagle.password=alex123
      ④启动
      	①查看/opt/kafka-eagle-web-1.3.7/bin/ke.sh是否有执行权限，如果没有，增加权限
      	②保证zookeeper集群和kafka集群已经启动
      	③使用命令启动：ke.sh start
      ⑤验证
      	jps，发现进程Bootstrap
      	登录http://192.168.8.105:8048/ke,默认账户：admin 密码：123456
```

### 企业开发补充点

##### kafka压力测试

```
企业中安装完kafka，需要使用kafka自带的脚本，对kafka进行压测，帮助发现CPU/内存/网络IO方面的瓶颈

具体步骤：
	①启动kafka集群，使用自定义群起脚本kafka-batch-start
	②测试生产者
		①运行命令：
            /opt/kafka_2.11-0.11.0.0/bin/kafka-producer-perf-test.sh
            --topic test  //向哪个主题生产消息
            --record-size 100	//每条信息大小，单位字节
            --num-records 50000 	//总共生产的信息数量
            --throughput 1000		//每秒生产多少条信息
            --producer-props bootstrap.servers=hadoop02:9092,hadoop03:9092,hadoop04:9092 //kafka集群地址
        ②结果分析：
            50000 records sent, 999.720078 records/sec (0.10 MB/sec), 9.59 ms avg latency, 251.00 ms max latency, 5 ms 50th, 29 ms 95th, 94 ms 99th, 213 ms 99.9th.
            
            平均每秒生产消息999.720078条
            峰值生产速度每秒100B*1000=0.1MB
            平均延时9.59毫秒
            最大延时251毫秒
            50%在5毫秒以内，45%在5< <29毫秒,4%在29< < 94毫秒，0.01%在94< < 213毫秒
	③测试消费者
		①运行命令：
            /opt/kafka_2.11-0.11.0.0/bin/kafka-consumer-perf-test.sh 
            --topic test 	//向哪个主题消费消息
            --fetch-size 10000 //每次消费的数量
            --messages 5000000 //消息的总数量
            --threads 1 //一个进程
            --zookeeper hadoop02:2181	//zookeeper集群地址
        ②结果分析：
start.time, end.time, data.consumed.in.MB, MB.sec, data.consumed.in.nMsg, nMsg.sec
2020-05-01 15:19:11:902, 2020-05-01 15:19:41:707, 14.4819, 0.4859, 151854, 5094.9170

	最大吞吐率 14.4819MB/秒
	平均每秒消费0.4859MB
	最大每秒消费151854条
	平均每秒消费5094.9170条
```

##### 计算kafka机器数量

```
Kafka机器数量（经验公式）=2*（峰值生产速度*副本数/100）+1
一般情况下，副本都是2，套用公式，一般3台机器可以满足需求
```

