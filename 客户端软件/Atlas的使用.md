# Atlas的使用

### 简介

```
Atlas是Hadoop的数据治理和元数据管理框架。
功能如下：
    Atlas支持各种Hadoop和非Hadoop元数据类型
    提供了丰富的REST API进行集成
    对数据血缘的追溯达到了字段级别
```

------

### 补充元数据

#### 元数据概念

​	元数据通常定义为”关于数据的数据”,在数据仓库中是定义和描述DW/BI系统的结构，操作和内容的所有信息。元数据贯穿了数据仓库的整个生命周期，使用元数据驱动数据仓库的开发，使数据仓库自动化，可视化。 

#### 元数据类型

```
1.业务元数据 
	业务元数据指从业务角度描述业务领域相关的概念、关系和规则的数据，包括业务术语和业务规则等信息。
2.技术元数据
	技术元数据指描述系统中技术细节相关的概念、关系和规则的数据，包括对数据结构、数据处理方面的描述，以及数据仓库、ETL、前端展现等技术细节方面的信息。

技术元数据又细分为：

       1.数据源元数据

        2.ETL元数据

       3.数据仓库元数据

       4.BI元数据

3.管理元数据
	管理元数据指描述管理领域相关的概念、关系和规则的数据，主要包括管理流程、人员组织、角色职责等信息。
```

#### 元数据架构

![元数据架构](D:\Git\MyRepository\Alex.blog.github.io\Images\元数据架构.png)

#### 元数据功能 

```
血缘分析：向上追溯元数据对象的数据来源。

影响分析：向下追溯元数据对象对下游的影响。

同步检查：检查源表到目标表的数据结构是否发生变更。

指标一致性分析：定期分析指标定义是否和实际情况一致。

实体关联查询：事实表与维度表的代理键自动关联
```

#### 元数据应用

```
ETL自动化管理：使用元数据信息自动生成物理模型，ETL程序脚本，任务依赖关系和调度程序。

数据质量管理：使用数据质量规则元数据进行数据质量测量。

数据安全管理：使用元数据信息进行报表权限控制。

数据标准管理：使用元数据信息生成标准的维度模型。

数据接口管理：使用元数据信息进行接口统一管理。
```

------

### 安装Atlas

```
步骤一：
	从官网http://atlas.apache.org/#/Downloads下载指定版本的源码包，我们使用
apache-atlas-0.8.4-sources.tar.gz
步骤二：编译源码与安装
	①将源码压缩包解压到window下D盘，修改pom.xml文件，修改：
		hbase、solor、zookeeper、kafka的版本与集群中相同
	②在D:\\apache-atlas-sources-0.8.4下进入命令行窗口，开始编译：
		mvn clean package -Pdist,embedded-hbase-solr
	注意：编译很大概率出错（有些镜像无法下载，需要vpn），这里直接使用编译好的版本apache-atlas-0.8.4-bin.tar.gz
	③将apache-atlas-0.8.4-bin.tar.gz上传到主机hadoop02下/opt目录，重命名atlas-0.8.4
步骤三：基本配置（必须）
	①集成hbase
		atlas本身集成了hbase，这里使用集群中的hbase-1.3.1版本
		①修改/opt/atlas-0.8.4/conf/atlas-application.properties:
			#atlas数据存放在zk集群
			atlas.graph.storage.hostname=hadoop02:2181,hadoop03:2181,hadoop04:2181
		②关联atlas与hbase,建立软连接：
			ln -s /opt/atlas-0.8.4/hbase/conf /opt/atlas-0.8.4/conf/hbase
		③修改/opt/atlas-0.8.4/conf/atlas-env.sh，添加：
			#添加hbase配置文件路径
			export HBASE_CONF_DIR=/opt/atlas-0.8.4/conf/hbase/conf
	②集成solr
		atlas本身集成了solr，这里使用集群中的solr-5.2.1版本
		①修改/opt/atlas-0.8.4/conf/atlas-application.properties:
			#solr的元数据存放在zk
			atlas.graph.index.search.solr.zookeeper-url=hadoop02:2181,hadoop03:2181,hadoop04:2181
		②将/opt/atlas-0.8.4/conf/solr目录拷贝到/opt/solr-5.2.1目录下，重命名为/opt/solr-5.2.1/atlas_conf
		③在solr中创建集合（可以理解为kafka中的主题）
			①启动solr，solr-batch.sh start
			②在创建三个集合
				bin/solr create -c vertex_index -d /opt/solr-5.2.1/atlas_conf -shards 3 -replicationFactor 2
				bin/solr create -c edge_index -d /opt/solr-5.2.1/atlas_conf -shards 3 -replicationFactor 2
				bin/solr create -c fulltext_index -d /opt/solr-5.2.1/atlas_conf -shards 3 -replicationFactor 2
			【注意】
			vertex_index、edge_index、fulltext_index是solr主题（类似kafka topic）
			-shards 指定该集合的分片数(类比kafka的分区数)
			-replicationFactor 指定每个分片的副本数（类比kafka的副本数）
			如果要删除集合，使用命令：
			bin/solr delete -c 集合名
	③集成kafka
		atlas本身集成了kafka，这里使用集群中的kafka_2.11-0.11.0.0版本
		①修改/opt/atlas-0.8.4/conf/atlas-application.properties:
#关闭内部通知
atlas.notification.embedded=false
#配置zk集群地址   
atlas.kafka.zookeeper.connect=hadoop02:2181,hadoop03:2181,hadoop04:2181
#配置kafka broker地址 
atlas.kafka.bootstrap.servers=hadoop02:9092,hadoop03:9092,hadoop04:9092
#默认400，延长kafka与zk的会话超时时间
atlas.kafka.zookeeper.session.timeout.ms=4000
#默认200，延长kafka与zk的连接超时时间
atlas.kafka.zookeeper.connection.timeout.ms=2000
#开启自动提交，消费者每次消费最新消息
atlas.kafka.enable.auto.commit=true
		②在kafka中创建指定的两个topic
			/opt/kafka_2.11-0.11.0.0/bin/kafka-topics.sh
			--zookeeper hadoop02:2181,hadoop03:2181,hadoop04:2181
			--create
			--replication-factor 3	//副本数
			--partitions 3		//分区数
			--topic _HOATLASOK  //指定主题
			
			/opt/kafka_2.11-0.11.0.0/bin/kafka-topics.sh
			--zookeeper hadoop02:2181,hadoop03:2181,hadoop04:2181
			--create
			--replication-factor 3
			--partitions 3
			--topic ATLAS_ENTITIES
	 ④集成hive
	 	①编辑/opt/atlas-0.8.4/conf/atlas-application.properties，添加：
            #atlas监控hive数据变化，并及时更新
            atlas.hook.hive.synchronous=false
            atlas.hook.hive.numRetries=3
            atlas.hook.hive.queueSize=10000
            atlas.cluster.name=primary
        ②配置atlas
        	①将配置文件添加到类加载器，进入/opt/atlas-0.8.4/hook/hive目录：
        	zip -u atlas-plugin-classloader-0.8.4.jar /opt/atlas-0.8.4/conf/atlas-application.properties 
        	②将配置关联到hive
        		cp /opt/atlas-0.8.4/conf/atlas-application.properties /opt/hive-1.2.1/conf/
        ③配置hive
        	①编辑/opt/hive-1.2.1/conf/hive-site.xml,添加：
                <!--关联hive和atlas-->
                <property>
                     <name>hive.exec.post.hooks</name>
                     <value>org.apache.atlas.hive.hook.HiveHook</value>
                </property>
            ②编辑/opt/hive-1.2.1/conf/hive-env.sh,添加：
            	#在 tez 引擎依赖的 jar 包后面追加 hive 插件相关 jar 包 export HIVE_AUX_JARS_PATH=/opt/module/hadoop-2.7.2/share/hadoop/common/hadoop-lzo-0.4.20.jar$TEZ_JARS,/opt/atlas-0.8.4/hook/hive/atlas-plugin-classloader-0.8.4.jar,/opt/atlas-0.8.4/hook/hive/hive-bridge-shim-0.8.4.jar
步骤四：其他配置
	①修改/opt/atlas-0.8.4/conf/atlas-application.properties:
            #atlas web服务地址
            atlas.rest.address=http://hadoop02:21000
            #是否在atlas服务启动后打开web页面
            atlas.server.run.setup.on.start=true

            #########  Entity Audit Configs  #########
            atlas.audit.hbase.tablename=apache_atlas_entity_audit
            atlas.audit.zookeeper.session.timeout.ms=1000
            atlas.audit.hbase.zookeeper.quorum=hadoop02:2181
	②开启日志功能
		默认atlas日志没有记录日志文件，编辑/opt/atlas-0.8.4/conf/atlas-log4j.xml:
		<appender name="perf_appender" class="org.apache.log4j.DailyRollingFileAppender">
            <param name="file" value="${atlas.log.dir}/atlas_perf.log" />
            <param name="datePattern" value="'.'yyyy-MM-dd" />
            <param name="append" value="true" />
            <layout class="org.apache.log4j.PatternLayout">
                <param name="ConversionPattern" value="%d|%t|%m%n" />
            </layout>
        </appender>

        <logger name="org.apache.atlas.perf" additivity="false">
            <level value="debug" />
            <appender-ref ref="perf_appender" />
        </logger>
步骤四：启动
	①依次重启hdfs、yarn、zk、kafka、hbase、solr集群
		start-dfs.sh
		start-yarn.sh
		zkServer-batch-start
		kafka-batch-start
		start-hbase.sh
		solr-batch.sh start
	②启动atlas服务
		/opt/atlas-0.8.4/bin/atlas_start.py
步骤五：导入元数据
	将hive中的元数据导入到atlas，使用命令：
		/opt/atlas-0.8.4/bin/import-hive.sh
步骤六：验证
	①jps出现Atlas进行
	②访问http://hadoop02:21000，用户名和密码都是admin
```

### 架构

![1593131028403](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1593131028403.png)