# HBASE的使用

### 安装配置

```
在主机hadoop02上安装：
 步骤一：
	从官网http://hbase.apache.org/downloads.html 下载相应版本的gz包，本机使用的hbase-1.3.1-bin.tar.gz
步骤二：
	使用sftp工具上传到VMWare主机hadoop02（192.168.8.105）的/opt目录并解压，重命名解压后的目录名称hbase-1.3.1
步骤三：配置信息
	①进入/opt/hbase-1.3.1,将当前目录配置到系统环境变量中：
        ①编辑/etc/profile文件，增加
                export HBASE_HOME=/opt/hbase-1.3.1
                export PATH=$PATH:$HBASE_HOME/bin
		②使配置立刻生效,source /etc/profile
		③分发 /etc/profile
	②修改/opt/hbase-1.3.1/conf/hbase-env.sh,
		#指定java运行环境
		export JAVA_HOME=/opt/jdk1.8.0_251
		#不使用hbase内置的zk
		export HBASE_MANAGES_ZK=false
	③修改/opt/hbase-1.3.1/conf/
	hbase-site.xml,
		<!--配置hbase的hdfs存储地址-->
			<property>
                <name>hbase.rootdir</name>
                <value>hdfs://hadoop02:9000/HBase</value>
            </property>
         <!--hbase开启集群模式-->
            <property>
                <name>hbase.cluster.distributed</name>
                <value>true</value>
            </property>
         <!-- hbase服务端口，0.98后的新变动，之前版本没有.port,默认端口为 60000 -->
            <property>
                <name>hbase.master.port</name>
                <value>16000</value>
            </property>
         <!--zk集群地址-->
            <property> 
                <name>hbase.zookeeper.quorum</name>
                 <value>hadoop02,hadoop03,hadoop04</value>
            </property>
         <!--zk的数据存放路径-->
            <property> 
                <name>hbase.zookeeper.property.dataDir</name>
                 <value>/opt/zookeeper-3.4.14/zkData</value>
            </property>
            
       ④配置群起脚本/opt/hbase-1.3.1/conf/regionservers,
       		hadoop02
       		hadoop03
       		hadoop04
       ⑤将hadoop的配置文件软连接（快捷方式）到hbase，使hbase能在自己的目录中访问到
       		ln -s /opt/hadoop-2.7.2/etc/hadoop/core-site.xml /opt/hbase-1.3.1/conf/core-site.xml
       		ln -s /opt/hadoop-2.7.2/etc/hadoop/hdfs-site.xml /opt/hbase-1.3.1/conf/hdfs-site.xml
步骤四：分发到hadoop03、hadoop04
	xsync /opt/hbase-1.3.1
步骤五：启动
	两种方式：
	方式一：单节点启动
		①启动master服务
			hbase-daemon.sh start master
		②启动regionserver服务
			hbase-daemon.sh start regionserver
	方式二：群起集群
		start-hbase.sh
步骤六：验证
	①登录192.168.8.105:16010,访问hbase监控界面
	②jps查看HRegionServer和HMaster进程
	③hbase shell，使用命令窗口
```

### Hbase架构

##### 概念

```
    Table，表，包含一个或多个Region
    Region，表的横向切片，包含一个或多个Store
    Column：等同于关系型数据库中的列
    Column family：列族（包含一个或多个列），表的纵向切片
    	列族常用的几个属性：
    		●NAME,列族名
    		●VERSIONS，版本数量，默认1
    		●MIN_VERSIONS,最小版本数，默认0，只有设置了TTL才会起作用
    		●TTL,存活时间，默认FOREVER，单位秒
    		●BLOCKSIZE,数据块大小，默认65536，单位kb
    		●BLOCKCACHE,块缓存，默认true
    		●COMPRESSION,压缩
    		●BLOOMFILTER,布隆过滤器
    Row key：行主键
	Store，保存Region中的一个列族的数据
	memStore，写缓存空间，存放着临时（还未flush写入StoreFile）的有序数据
	StoreFile，数据实际存放在hdfs上的物理文件，StoreFile中的数据是有序存放（按照Row key字典排序）
```

##### 物理存储结构

![Hbase物理存储结构](D:\Git\MyRepository\Alex.blog.github.io\Images\Hbase物理存储结构.png)

##### 逻辑架构

![Hbase架构](D:\Git\MyRepository\Alex.blog.github.io\Images\Hbase架构.png)

### Hbase缓存机制

```
HBase在实现中提供了两种缓存结构：MemStore和BlockCache。
MemStore
1、其中MemStore称为写缓存
2、HBase执行写操作首先会将数据写入MemStore，并顺序写入HLog，这种设计可以极大地提升HBase的写性能。
3、MemStore对于读性能也至关重要，假如没有MemStore，读取刚写入的数据就需要从文件中通过IO查找，这种代价显然是昂贵的！

BlockCache
1、BlockCache称为读缓存
2、HBase会将一次文件查找的Block块缓存到Cache中，以便后续同一请求或者邻近数据查找请求，可以直接从内存中获取，避免昂贵的IO操作。
```

### Hbase Shell基本语法

**一：DDL操作：**

- 进入hbase客户端 hbase shell

- 查看当前用户表 list

- 创建命名空间 create_namespace '命名空间名'

- 创建表
              语法①：create '命名空间:表名','列族1','列族2',...

  ​		语法②：create '命名空间:表名', {NAME => '列族1', VERSIONS => 5,..}, ...

- 修改表

  ​		①修改列族属性

  ​				alter '表名',{列族属性1 => '新的值',...}

  ​		②删除列族

  ​				alter '表名',{ NAME => '列族名', METHOD => 'delete'}		
  
- 删除表

     ​		先更改表的状态为disable：disable '表名'

     ​		再drop：drop '表名'

**二：DML操作**

- 查看表结构，describe '表名'

- 扫描查看表数据

  ​		语法①：scan '命名空间:表名'

  ​		语法②：scan '表名',{COLUMNS => ['列族1:列名1','列族2:列名2'],LIMIT => n,STARTROW => 'Row key',ENDROW => 'Row key'}

- 查看指定数据

  ​		语法①：get '表名','Row key'

  ​		语法②：get '表名','Row key','列族:列名'

- 更新指定字段数据（不能更新多列）

  ​		put '表名','Row key','列族:列名','新的列值'

- 插入数据（不能插入多列）

  ​		put '表名','Row key','列族:列名','列的值'

- 删除数据

  ​		①删除对应Row key的数据

  ​				deleteall '表名','Row key'

  ​		②删除某Row key的某一列数据

  ​				delete '表名','Row key','列族:列名'

- 清空表数据

  ​		先disable，disable '表名'

  ​		再truncate，truncate '表名'

  

  **三：tools**

  ​	①flush，手动将memStore中的缓存数据写入StoreFile

  ​			默认情况下，当memstore大小达到hbase.hregion.memstore.flush.size=128M才会将一个Region中的memstore刷到storeFile，手动flush可以人为控制刷写的时机，生产中尽量不用手动flush

  ​    ②balancer	

  ​	③major_compact，手动将同一个Region中的所有StoreFile合并成一个

  ​	④merge

  ### Hbase+MR

  ##### 简介

  ```
  通过MR操作Hbase，可以实现数据从本地文件系统导入到Hbase，也可以从Hbase表读出数据使用MR做数据分析
  ```

  ##### 配置

  ```
  ①查看HBASE_CLASSPATH的环境变量
  	在/opt/hbase-1.3.1/bin目录下，使用命令：./hbase mapredcp
  ②将上面的环境变量添加到HADOOP_CLASSPATH中，使MR能调用到HBase的jar包
  	在/opt/hadoop-2.7.2/etc/hadoop/hadoop-env.sh增加
  		export HADOOP_CLASSPATH=$HADOOP_CLASSPATH:/opt/hbase-1.3.1/lib/hbase-protocol-1.3.1.jar:/opt/hbase-1.3.1/lib/guava-12.0.1.jar:/opt/hbase-1.3.1/lib/netty-all-4.0.23.Final.jar:/opt/hbase-1.3.1/lib/hbase-client-1.3.1.jar:/opt/hbase-1.3.1/lib/metrics-core-2.2.0.jar:/opt/hbase-1.3.1/lib/hbase-prefix-tree-1.3.1.jar:/opt/hbase-1.3.1/lib/htrace-core-3.1.0-incubating.jar:/opt/hbase-1.3.1/lib/hbase-common-1.3.1.jar:/opt/hbase-1.3.1/lib/zookeeper-3.4.6.jar:/opt/hbase-1.3.1/lib/protobuf-java-2.5.0.jar:/opt/hbase-1.3.1/lib/hbase-hadoop-compat-1.3.1.jar:/opt/hbase-1.3.1/lib/hbase-server-1.3.1.jar
  ```

  ##### 官方案例一: 统计hbase中表的记录

  ```
  需求分析：
  	统计hbase中new_table的记录数
  具体实现：
  	①启动hdfs集群和yarn集群
  	②进入目录/opt/hadoop-2.7.2/bin，使用命令：
  		yarn jar /opt/hbase-1.3.1/lib/hbase-server-1.3.1.jar rowcounter new_table
  ```

  ##### 官方案例二: 将hdfs导入到hbase表

  ```
  需求分析：
  	①将hdfs://hadoop02:9000/usr/input/test.tsv中的数据导入到hbase test表
  具体实现：
  	①主机hadoop02的linux文件系统创建test.tsv,编辑内容：
        rk100  laoli   50
        rk101  wang    55
        rk102  deng    48
  	②将test.tsv上传到hdfs://hadoop02:9000/usr/input/test.tsv
  	③通过hbase shell创建test表：create 'test','cf1'
  	④启动hdfs集群和yarn集群
  	⑤使用命令：
  		/opt/hadoop-2.7.2/bin/yarn jar /opt/hbase-1.3.1/lib/hbase-server-1.3.1.jar importtsv -Dimporttsv.columns=HBASE_ROW_KEY,cf1:name,cf1:age test hdfs://hadoop02:9000/usr/input
  ```

  ### Hbase+Hive

  ##### 简介

  ```
  Hive操作Hbase，可以实现比较复杂的统计分析
  ```

  ##### 配置

  ```
  ①在Hive的lib包以软连接的方式增加Hbase的jar包（以下8个）：
  	ln -s /opt/hbase-1.3.1/lib/hbase-common-1.3.1.jar /opt/hive-1.2.1/lib/hbase-common-1.3.1.jar
  	ln -s /opt/hbase-1.3.1/lib/hbase-server-1.3.1.jar /opt/hive-1.2.1/lib/hbase-server-1.3.1.jar
  	ln -s /opt/hbase-1.3.1/lib/hbase-client-1.3.1.jar /opt/hive-1.2.1/lib/hbase-client-1.3.1.jar
  	ln -s /opt/hbase-1.3.1/lib/hbase-protocol-1.3.1.jar /opt/hive-1.2.1/lib/hbase-protocol-1.3.1.jar
  	ln -s /opt/hbase-1.3.1/lib/hbase-it-1.3.1.jar /opt/hive-1.2.1/lib/hbase-it-1.3.1.jar
  	ln -s /opt/hbase-1.3.1/lib/htrace-core-3.1.0-incubating.jar /opt/hive-1.2.1/lib/htrace-core-3.1.0-incubating.jar
  	ln -s /opt/hbase-1.3.1/lib/hbase-hadoop2-compat-1.3.1.jar /opt/hive-1.2.1/lib/hbase-hadoop2-compat-1.3.1.jar
  	ln -s /opt/hbase-1.3.1/lib/hbase-hadoop-compat-1.3.1.jar /opt/hive-1.2.1/lib/hbase-hadoop-compat-1.3.1.jar
  
  ②在/opt/hbase-1.3.1/conf/hive-site.xml增加对zookeeper的配置：
  	    <!-- 配置zookeeper集群地址 -->
              <property>
                  <name>hive.zookeeper.quorum</name>
                  <value>hadoop02,hadoop03,hadoop04</value>
              </property>
          <!-- zookeeper服务端口 -->
              <property>
                  <name>hive.zookeeper.client.port</name>
                  <value>2181</value>
              </property>
  ③分发到hadoop03、hadoop04
  	xsync /opt/hbase-1.3.1/lib/
  	xsync /opt/hbase-1.3.1/conf/hive-site.xml
  ```

  ##### 案例一：同步操作Hive表和Hbase表

  ```
  需求分析：
  	①在Hive建立hive_hbase_staff表，要求在Hbase中自动创建hbase_staff表
  	②Hive对表hive_hbase_staff的操作，也会同步到Hbase中的hbase_staff表
  	注意：
  		实际上，数据是存储在hbase，hive中hive_hbase_staff表只是映射到了Hbase中的hbase_staff表，操作任意一个表，数据都可以实现同步
  ```
  
```
  具体实现：
  	①在Hive中创建hive_hbase_staff表，同时关联Hbase中hbase_staff表
  		CREATE TABLE hive_hbase_staff(
  			id int,
  			name string,
  			salary int,
  			position string
  		) STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
  		WITH SERDEPROPERTIES ("hbase.columns.mapping" = 
          ":key,cf1:name,cf1:sal,cf1:pos")
          TBLPROPERTIES ("hbase.table.name" = "hbase_staff");
       ②第一步有可能会报错，原因是hive和hbase版本可能不兼容
       	解决办法：
       		①从官网下载源码apache-hive-1.2.2-src.tar.gz，将其中的子工程hbase-handler导入到eclipse中
       		②为hbase-handler工程添加好jar依赖后重新export到hive-hbase-handler-1.2.1.jar
       		③将新的hive-hbase-handler-1.2.1.jar替换/opt/hive-1.2.1/lib/hive-hbase-handler-1.2.1.jar
       ③验证：
       	①在hive和hbase分别看到hive_hbase_staff表、hbase_staff表
       	②在hive中插入数据，同时看到hbase也被插入了数据
```

  ##### 案例二：Hive操作Hbase已经存在的表

  ```
需求分析：
  	①在Hive创建一个外部表hive_hbase_external_staff，关联Hbase已经存在的表hbase_staff
	②Hive对表hive_hbase_staff的操作，也会同步到Hbase中的hbase_staff表
  ```

  ```
  具体实现：
  	①在Hive中创建外部表
  		CREATE EXTERNAL TABLE hive_hbase_external_staff(
  			id int,
  			name string,
  			salary int,
  			position string
  		) STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
  		WITH SERDEPROPERTIES ("hbase.columns.mapping" = 
          ":key,cf1:name,cf1:sal,cf1:pos")
          TBLPROPERTIES ("hbase.table.name" = "hbase_staff");
       ②验证：
       	①在Hive表hive_hbase_external_staff能查询到Hbase中hbase_staff的所有数据
       	②Hive对表hive_hbase_staff的操作，也会同步到Hbase中的hbase_staff表
  ```

  ### Hbase高可用

  ##### 简介

  ```
  在 HBase 中 HMaster 负责监控 HRegionServer 的生命周期，均衡 RegionServer 的负载，如果 HMaster 挂掉了，那么整个 HBase 集群将陷入不健康的状态，并且此时的工作状态并不会维持太久。Hbase支持HMaster的高可用，只需要增加简单的配置。
  ```

  ##### 配置

  ```
  ①在/opt/hbase-1.3.0/conf下新增backup-masters文件，编辑：	
  	hadoop03
  	hadoop04
  ②分发给hadoop03、hadoop04
  	xsync backup-masters
  ③验证：
  	①在主机hadoop02上开启hbase集群，观察hadoop02是集群Master，hadoop03和hadoop04作为集群的Backup Master也被启动了
  	②杀掉hadoop02上的HMaster进程
  	③观察hadoop03或hadoop04成为了新的Master
  补充：
  	hbase依赖zookeeper完成了Master的重新投票选举
  ```

  ### Hbase预分区

  ##### 简介：为什么要预分区

  ```
  建表时默认只有一个Region，随着数据的不断写入，Region 会自动进行拆分。Region Split策略：
  	①在Hbase 0.94版本之前
  		当Region中一个Store下所有StoreFile总大小超过hbase.hregion.max.filesize时一分为二
  	②在Hbase0.94之后
  		当Region中一个Store下所有StoreFile总大小超过： 
  		Min(R^2 * hbase.hregion.memstore.flush.size,hbase.hregion.max.filesize)
  	注意：
  		hbase.hregion.max.filesize默认大小10737418240（10Gb）
  		hbase.hregion.memstore.flush.size是指memStore的缓存空间，默认大小134217728（128Mb）
  		R为表中Region的数量
  		
  以下演示Hbase0.94之后 Region分区的过程：
  	第1次分区：当数据达到1^1 * 128M = 128M，
  		R1 = 64M,R2 = 64M
  	第2次分区：当其中一个Region数据达到2^2 * 128M = 512M，
  		R1 = 64M,R2 = 256M,R3 = 256M
  	第3次分区:当其中以一个Region数据达到3^3 * 128M = 1152M,
  		R1 = 64M,R2 = 256M,R3 = 576M,R4 = 576M
      第4次分区:当其中以一个Region数据达到4^4 * 128M = 2048M,
  		R1 = 64M,R2 = 256M,R3 = 576M,R4 = 1024M,R5 = 1024M
  	第5次分区:当其中以一个Region数据达到5^5 * 128M = 3200M,
  		R1 = 64M,R2 = 256M,R3 = 576M,R4 = 1024M,R5 = 1600M,R6 = 1600M
  	第6次分区:当其中以一个Region数据达到6^6 * 128M = 4608M,
  		R1 = 64M,R2 = 256M,R3 = 576M,R4 = 1024M,R5 = 1600M,R6 = 2304M,R7 = 2304M
  	第7次分区:当其中以一个Region数据达到7^7 * 128M = 6272M,
  		R1 = 64M,R2 = 256M,R3 = 576M,R4 = 1024M,R5 = 1600M,R6 = 2304M,R7 = 3136M,R8 = 3136M
  	第8次分区:当其中以一个Region数据达到8^8 * 128M = 8192M,
  		R1 = 64M,R2 = 256M,R3 = 576M,R4 = 1024M,R5 = 1600M,R6 = 2304M,R7 = 3136M,R8 = 4096M,R9 = 4096M
  	第9次分区:当其中以一个Region数据达到9^9 * 128M = 10368M,
  		R1 = 64M,R2 = 256M,R3 = 576M,R4 = 1024M,R5 = 1600M,R6 = 2304M,R7 = 3136M,R8 = 4096M,R9 = 5184M,R10 = 5184M
  
  不难发现：R1~R10分区大小差异很大，带来热点问题：大量的数据都集中到一个RegionServer，查询效率极低
  ```

  ##### 常见的预分区

```
①手动设定预分区
	create '表名','列族名',SPLITS => ['1000','2000','3000']
②生成16进制序列预分区
	create '表名','列族名',{NUMREGIONS => 分区数n, SPLITALGO => 'HexStringSplit'}
③按照文件中设置的规则预分区
	①在/opt/hbase-1.3.0下创建splits.txt文件，编辑：
		region1
		region2
		region3
	②执行：create '表名','列族名',SPLITS_FILE => 'splits.txt'
④在API中创建预分区（生产中常用）
	//自定义算法，产生一系列 hash 散列值存储在二维数组中
    byte[][] splitKeys = 某个散列值函数
    //创建 HbaseAdmin 实例
    HBaseAdmin hAdmin = new HBaseAdmin(HbaseConfiguration.create());
    //创建 HTableDescriptor 实例
    HTableDescriptor tableDesc = new HTableDescriptor(tableName);
    //通过 HTableDescriptor 实例和散列值二维数组创建带有预分区的 Hbase 表
    hAdmin.createTable(tableDesc, splitKeys);
```

### Hbase优化

1．允许在 HDFS 的文件中追加内容 

hdfs-site.xml、hbase-site.xml 

```
属性：dfs.support.append 

解释：开启 HDFS 追加同步，可以优秀的配合 HBase 的数据同步和持久化。默认值为 true。 
```

2．优化 DataNode 允许的最大文件打开数 

hdfs-site.xml 

```
属性：dfs.datanode.max.transfer.threads 

解释：HBase 一般都会同一时间操作大量的文件，根据集群的数量和规模以及数据动作，设置为 4096 或者更高。默认值：4096 
```

3．优化延迟高的数据操作的等待时间 

hdfs-site.xml 

```
属性：dfs.image.transfer.timeout 

解释：如果对于某一次数据操作来讲，延迟非常高，socket 需要等待更长的时间，建议把该值设置为更大的值（默认 60000 毫秒），以确保 socket 不会被 timeout 掉。 
```

4．优化数据的写入效率 

mapred-site.xml 

```
属性： 
mapreduce.map.output.compress 
mapreduce.map.output.compress.codec 

解释：开启这两个数据可以大大提高文件的写入效率，减少写入时间。第一个属性值修改为true，第二个属性值修改为：org.apache.hadoop.io.compress.GzipCodec 或者其他压缩方式。
```

 5．设置 RPC 监听数量 

hbase-site.xml 

```
属性：Hbase.regionserver.handler.count 

解释：默认值为 30，用于指定 RPC 监听的数量，可以根据客户端的请求数进行调整，读写请求较多时，增加此值。 
```

6．优化 HStore 文件大小 

hbase-site.xml 

```
属性：hbase.hregion.max.filesize 

解释：默认值 10737418240（10GB），如果需要运行 HBase 的 MR 任务，可以减小此值，因为一个 region 对应一个 map 任务，如果单个 region 过大，会导致 map 任务执行时间过长。该值的意思就是，如果 HFile 的大小达到这个数值，则这个 region 会被切分为两个 Hfile。 
```

7．优化 HBase 客户端缓存 

hbase-site.xml 

```
属性：hbase.client.write.buffer 

解释：用于指定 Hbase 客户端缓存，增大该值可以减少 RPC 调用次数，但是会消耗更多内存，反之则反之。一般我们需要设定一定的缓存大小，以达到减少 RPC 次数的目的。 
```

8．指定 scan.next 扫描 HBase 所获取的行数 

hbase-site.xml 

```
属性：hbase.client.scanner.caching 

解释：用于指定 scan.next 方法获取的默认行数，值越大，消耗内存越大。 
```

9．flush、compact、split 机制 

当 MemStore 达到阈值，将 Memstore 中的数据 Flush 进 Storefile；compact 机制则是把 flush 

出来的小文件合并成大的 Storefile 文件。split 则是当 Region 达到阈值，会把过大的 Region 

一分为二。 