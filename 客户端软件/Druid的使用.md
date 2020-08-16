# Druid的使用

### 简介

```
Druid是一个开源、分布式、面向列存储的OLAP实时数据存储和分析系统。
特点：高可用，数据吞吐量大，支持流式数据源，查询速度快（亚秒级），集群动态扩展
缺点：支持数据源比较少，目前只有kafka
```

### 安装

```
步骤一：下载
	官网https://imply.io/下载相应版本，本机使用imply-2.7.10.tar.gz
步骤二：
	使用sftp工具上传到VMWare主机hadoop02（192.168.8.105）的/opt目录并解压，重命名解压后的目录名称/opt/imply-2.7.10
步骤二：配置
	①配置zk集群地址
		①修改配置文件/opt/imply-2.7.10/conf/druid/_common/common.runtime.properties：
			druid.zk.service.host=hadoop02:2181,hadoop03:2181,hadoop04:2181
		②不使用内置zk
			修改/opt/imply-2.7.10/conf/supervise/quickstart.conf：
				:verify bin/verify-java
                #:verify bin/verify-default-ports
                #:verify bin/verify-version-check
                :kill-timeout 10
                #!p10 zk bin/run-zk conf-quickstart
步骤四：启动
	①启动zk集群
		zkServer-batch-start
	③启动imply
		bin/supervise -c conf/supervise/quickstart.conf
步骤五：验证
	访问hadoop02:9095/datasets
```

### Druid框架原理![1594272169034](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1594272169034.png)

### 对比Presto/Spark SQL/Kylin/Elasticsearch

![计算框架对比图](D:\Git\MyRepository\Alex.blog.github.io\Images\计算框架对比图.png)

### 案例

```
需求：
	监控kafka 集群topic_start主题的实时数据变化，并进行实时sql查询
实现：
	①启动kafka集群
		kafka-batch-start
	②开启flume日志采集服务
		file-flume-kafka.sh
	③模拟产生日志(时间间隔设置长一些)
		log_collector.sh 1000 500
	④启动imply，登录hadoop02:9095/datasets
		①新增kafka数据源
		②配置kafka broker集群地址：hadoop02:9092,hadoop03:9092,hadoop04:9092，以及数据来自哪个主题,这里只能使用topic_start（因为topic_event不是标准的json数据）
		③“sample adn continue”，下一步
		④指定时间戳字段（秒或者毫秒），下一步
		⑤指定查询的字段，一般只保留需要的字段（字段少可以减少预计算的时间开销）
		⑥最后，等待预计算的过程结束就能使用sql进行实时查询
```

