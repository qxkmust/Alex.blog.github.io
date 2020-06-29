# HIVE的使用

### 下载安装

```
步骤一：
	从官网http://hive.apache.org/downloads.html下载相应版本的gz包，本机使用的apache-hive-1.2.1-bin.tar.gz
步骤二：
	使用sftp工具上传到VMWare主机hadoop02（192.168.8.105）的/opt目录并解压，重命名解压后的目录名称hive-1.2.1
步骤三：配置信息
	①进入/opt/hive-1.2.1/bin,将当前目录配置到系统环境变量中：
        ①编辑/etc/profile文件，增加
                export HIVE_HOME=/opt/hive-1.2.1
                export PATH=$PATH:$HIVE_HOME/bin
		②使配置立刻生效,source /etc/profile
	②进入/opt/hive-1.2.1/conf，重命名hive-env.sh.template为hive-env.sh，编辑：
		export HADOOP_HOME=/opt/hadoop-2.7.2 #如果系统环境变量中已经配置了HADOOP_HOME，可以省略这个配置
		export HIVE_CONF_DIR=/opt/hive-1.2.1/conf
步骤四：启动hive服务
	使用命令hive
```

### 一些重要配置

```
①修改hive.log的存放目录，默认是放到/tmp/root/下
	进入/opt/hive-1.2.1/conf,拷贝hive-log4j.properties.template，重命名为hive-log4j.properties，修改hive.log.dir=/opt/hive-1.2.1/logs
②修改hive的数据仓库路径，默认是在hdfs://user/hive/warehouse
	①进入/opt/hive-1.2.1/conf,复制hive-default-xml.template重命名为hive-site.xml
		cp hive-default-xml.template hive-site.xml
	②修改hive-site.xml，
		<property>
			<name>hive.metastore.warehouse.dir</name>
			<value>/usr/alex/hive</value>
		</property>
③使用hql查询表数据，默认不显示当前库以及表头信息（字段），需要在hive-site.xml增加配置：
		<property>
			<name>hive.cli.print.header</name>
			<value>true</value>
		</property>
		<property>
			<name>hive.cli.print.current.db</name>
			<value>true</value>
		</property>
```

### 数据类型

| hive基本数据类型 | java数据类型 | 长度         |
| ---------------- | ------------ | ------------ |
| TINYINT          | byte         | 1byte        |
| SMALINT          | short        | 2byte        |
| INT              | int          | 4byte        |
| BIGINT           | long         | 8byte        |
| BOOLEAN          | boolean      | true或false  |
| FLOAT            | float        | 单精度浮点数 |
| DOUBLE           | double       | 双精度浮点数 |
| STRING           | string       | 字符串       |
| TIMESTAMP        |              | 时间类型     |

| hive集合数据类型 | 描述            | 语法                 |
| ---------------- | --------------- | -------------------- |
| STRUCT           | 类型C语言struct | {键:值,键:值}        |
| MAP              | 类似java的MAP   | {键:值,键:值}        |
| ARRAY            | 类似java的ARRAY | [相同类型的数据集合] |

注意：复杂数据类型不能通过insert语句插入，只能通过load data local inpath '数据文件路径' into table 表名 [partition(分区字段='分区名')]

### HIVE的交互命令

```
①hive -d key=value,定义变量
	将某个路径定义到变量，可以在hive客户端使用这个变量
②hive -e "sql语句"
③hive -f sql文件路径，启动hive客户端后立刻执行sql文件中的sql语句，之后推出客户端（一般使用定时任务将执行结果输出到文件，用于第二天分析）
④hive -i sql文件路径，启动hive客户端后立刻执行sql文件中的sql语句，不退出客户端
⑤hive --hivevar key=value,同①定义变量
⑥hive -help，查看帮助
```

### HQL的使用

```
库操作：
    ①创建库，create database if not exists mydb
    			comment '数据库说明'
             location '/usr/alex/hive/mydb';   //指定存放在hdfs上的路径
            
    ②删除库，drop database mydb cascade，（如果mydb中有表，需要加cascade级联删除）
表操作：
    ①创建表，包括表字段，以及指定数据文件在读取时的切分规则
        create table if not exists college(
            name string,
            city string,
            is_struct struct<name:string,city:string>,
            is_map map<string,string>,
            is_array array<string>
        ) 
        comment '表描述'							//非必须
        partitioned by (country string)			//非必须，按照列country分区（存放不同的文件夹）
        clustered by (city string)
        into 4 buckets							//非必须，按照已存在的字段city分4个桶（存放到四个数据块）
        row format delimited fields terminated by ','        //非必须，行字段按照','分割
        collection items terminated by '_' 		//非必须，集合（struct,map,array）按照'_'分割
        map keys terminated by ':'				//非必须，map中的键值对按照':'分割
        location '/usr/alex/hive/mydb/college'	//非必须，指定表文件的存放目录
        stored as textfile;		//非必须，指定存储格式，默认textfile
    ③将外部文件数据导入到表中 
        ①将以下内容写到/opt/hive-1.2.1/mydb/college文件中：
            字段name,字段city,struct属性name值_struct属性city值,map属性name:map属性name值_map属性city:map属性city值,array值1_array值2:array值3
        ②使用命令：load data local inpath '/opt/hive-1.2.1/mydb/college' into table college;
        命令格式：
        	load data 【local】 inpath '数据文件所在路径' into table 表名 【partition(分区字段='分区名')】
        注意：如果不使用【local】默认从HDFS加载数据，相当于mv操作
     ②删除表，drop table if exists 表名
     ③清空表，truncate table 表名
     ④表分区操作
     	①在创建表的时候，指定分区的列
     	②分区表建立之后，每次插入数据必须指定到具体的分区
     		insert into/overwrite（先清除原来的数据，再插入） table 表名(字段1,字段2) partition(分区字段=分区名) values(值1,值2)
     	③添加分区，alter table college add partition(分区字段='分区名')
     	④删除分区，alter table college drop partition(分区字段='分区名')
     ⑤复制表
     	①复制表结构
     		create table 表2 like 表1，复制表1的结构给表2
     	②复制表结构和数据
     		create table 表2 as select * from 表1，复制表1的结构和数据给表2
列操作：
     	①新增列
     		alter table 表名 add columns (列名1 类型,列名2 类型)
     	②重置列（类似删除列的功能）
     		alter table 表名 replace columns(列名1 类型,列名2 类型,...)
数据导入/导出：
	①从本地/HDFS导入数据到表
        load data 【local】 inpath '数据文件所在路径' 【overwrite覆写】 into table 表名 【partition(分区字段='分区名')】
        注意：不加local就是导入到HDFS，不加overwrite就是追加
	②将表数据格式化后导出到本地/HDFS（不加local就是导入到HDFS）
		insert overwrite 【local】 directory '路径' 
		ROW FORMAT DELIMETED FIELDS TERMINATED BY '\t'
		select 语句；
		
		
Hive借鉴了mysql和oracle的大部分语法：
	hive不支持ifnull，只能用if(逻辑表达式,为真时值1，为假时值2)
	cast(字段 as 目标类型)，将字段或者常量转换为目标类型
	nvl(字段,缺省值)，设置缺省值，同oracle用法，mysql用法为ifnull
	unix_timestamp()，获取当前时间戳，参数也可以指定日期时间类型字符串
	from_unixtime(时间戳)，获取指定时间戳对应的日期
	date_format(日期类型字符串,'格式') 格式：yyyy-MM-dd HH:mm:ss 或者 yyyy-MM-dd hh:mm:ss
	date_sub(日期类型字符串,n)，按照给定日期，往前推n天的日期
	date_add(日期类型字符串,n)，按照给定日期，往后推n天的日期
	dayofmonth(日期类型字符串)，按照给定日期，返回是本月的第几天
	last_day(日期类型字符串),按照给定日期，返回月底那天
	next_day(日期类型字符串,'MO'/'TU'/'WE'/'TH'/'FR'/'SA'/'SU'),按照定义日期，往后找周几
	collect_set(基本数据类型字段)，将某字段的值进行去重汇总，结果放到一个array中
	concat(多个基本数据类型字段),与mysql语法类似
	split(string字段,'分隔符')，分割字符串
	concat_ws('分隔符',string类型字段1，string类型字段2)，只能拼接string类型或array<string>字段
	explode(array类型或map类型字段)，可以把属性拆分（与第一关系型数据库的第一范式相悖）
	lateral view(侧视图)，与炸裂函数配置使用：
		【语法】
		select 临时字段1 from 表 lateral view explode(表中的Array类型字段1) 临时表1 as 临时字段1
		select 临时字段1,临时字段2 from 表 lateral view explode(表中的Map类型字段1) 临时表1 as 临时字段1,临时字段2  ，将Map的key和value分别赋值给临时字段1,临时字段2
		
		
	窗口函数over()可以指定数据块的大小，针对数据块使用聚合函数（SUM,COUNT,MIN,MAX,AVG,RANK）进行统计
	用法：
		sum(字段1) over(),统计字段1的所有数据
		sum(字段1) over(cluster by 字段2)，按照字段2分区，分别统计区内字段1
		sum(字段1) over(partition by 字段2 sort by 字段3)，按照字段2分区，分别统计区内字段1，区内按字段3排序
		
	以下演示窗口函数over():
	bill表包含最近的超市消费记录，有以下数据：
	##############################################
	消费者				日期				金额	
    name			  time			  cost
    大力				2020-01			200.0
    金巧巧				2020-20			100.0
    大力				2020-01			200.0
    大力				2020-08			400.0
    大力				2020-15			600.0
    金巧巧				2020-07			400.0
    金巧巧				2020-14			200.0
    大柱				2020-02			100.0
	##############################################
	需求：
		①统计每个人当日消费的总金额，清单按日期升序
			select *,SUM(cost) over(distribute by name sort by time) from bill;
			或者
			select *,SUM(cost) over(partition by name order by time) from bill;
		②统计每个人最近消费的总金额，清单按日期升序
			select *,SUM(cost) over(distribute by name) from (select * from bill order by time) bill;
		③统计每个人最近消费记录，及上一次购买时间
			select *,LAG(time,1,'1970-01-01') over(distribute by name sort by time) from bill;
			注意：
				①LAG(字段,n,缺省值)，获取字段往前第n条记录的值
				②LEAD(字段,n,缺省值)，获取字段往后第n条记录的值
				
排名函数rank:
使用语法：
	①rank()，排序相同时会重复
	②dense_rank()，排序相同时会重复
	③row_number()，按行统计
	
需求：
	①统计每个人最近消费的记录，打印序号
		select *,row_number() over(partition by name order by time) from bill;
	②统计每个人最近消费的记录，相同记录（日期和金额相同）共用一个序号
		select *,dense_rank() over(partition by name order by time) from bill;
	③统计每个人最近消费的记录，相同记录公用一个序号，总记录数不变
		select *,rank() over(partition by name order by time) from bill;

工作中更多函数，可查看帮助：
	①show functions，查看所有函数名
	②show functions like '*关键字*'
 	③desc function extended 函数名，查看某个函数的详细用法
```

###  HQL常见语法错误

```
【错误1】cannot recognize input near '<EOF>' '<EOF>' '<EOF>' in subquery source
【原因】使用子查询的结果作为查询表，必须为该表起别名
```

### 自定义函数UDF

```
即使是炸裂函数如explode也无法解析出复杂的类型，需要用户自定义函数(UDF)
用户自定义函数UDF(User Defined Function)包含以下几种：
		①UDF，一进一出
        ②UDAF，多进一出（例如Hive聚合函数，sum/avg/min/max/rank/count）
        ③UDTF，一进多出（例如Hive的炸裂函数explode）
     注意：
        ①一进一出，是指一行记录（可以包含多个字段）作为输入，产生一个结果
        ②多进一出，是指多行记录（可以只有一个字段）作为输入，产生一个结果
        ③一进多出，是指一行记录作为输入，产生多行结果
 
HIVE支持用户自定义函数(UDF)，具体操作：
	①IDEA创建Maven工程，在pom.xml文件注入依赖：
			<dependency>
                <groupId>org.apache.hive</groupId>
                <artifactId>hive-exec</artifactId>
                <version>1.2.1</version>
            </dependency>
    ②主类MyUDF继承org.apache.hadoop.hive.ql.exec.UDF类，重写evaluate方法（可重载）
    	注意：
    		①方法名只能是evaluate
    		②通过重载的方式声明多个evaluate
    		③evaluate方法是在hive的自定义函数被使用时才回调
    		④hive中自定义函数通过传入参数的个数，回调不同的evaluate方法
    ③将Maven工程打包，上传到linux文件系统hive的工作目录的/lib下
    ④创建函数
    	①使用命令hive登录hive客户端
    	②使jar立刻生效（将jar加入classpath）
    		add jar jar包的全路径    		
    	③创建函数
    		create function 函数名(随意) as "MyUDF的全类名";
```

### HIVE的使用

```
介绍：
	Hive是基于Hadoop的一个数据仓库工具，可以将结构化的数据文件映射为一张表，主要帮助我们使用HQL（hive query language）分析已经存在在HDFS上的大量数据。
架构：
```

![hive架构](D:\Git\MyRepository\Alex.blog.github.io\Images\hive架构.png)

hive的数据库是存放在hdfs上，原理是将hql转化为MR程序，所以使用hive前要在hadoop02上开启单机版hdfs和yarn服务

```
derby的使用：
	hive默认使用derby数据库（java开发的轻量数据库，大小2M）存储数据库文件（不包括数据，只存数据库/表结构）
	①在/opt/hive-1.2.1下创建/mydb目录，进入/opt/hive-1.2.1/mydb,开启hive客户端
	②hive默认会在当前目录下创建metastore_db目录（derby的默认数据存放路径）
	②hive的hql类似sql，但是只能进行增查
		①创建数据库
			create datatable mydb;
		②切换数据库
			use mydb;
		③创建表stu
			create table stu(id int,name varchar(20)) row format delimited fields terminated by '\t'
			表明使用"\t"对表字段进行分割，默认是符号^A,在vi编辑模式下通过ctrl+v,ctrl+a打出来
		④插入数据
			insert into stu values(1,'lessi'),(2,'sasa');
		⑥查询数据
			select * from stu;
	④以上derby数据库存放在hdfs的目录/user/hive/warehouse/mydb.db/stu中，可以通过向hdfs的数据库/表的对应目录中上传文件，实现表数据新增
		①将hadoop02主机上/opt下的文件data.txt，内容：
                3\tdavid
                4\tlucy
			上传到hdfs上，hadoop fs -put /opt/data.txt /user/hive/warehouse/mydb.db/stu
		②登录hive客户端，发现stu表数据新增了两条
	
derby存在的缺陷：
	①不支持远程连接
	②不支持多用户多线程
```

替换derby为mysql

```
开发中不会使用derby，需要替换为mysql，方便远程连接，这里使用mysql5.5.52(mysql8与hive1兼容不好)
	①驱动包mysql-connector-java-5.1.47.jar拷贝到/opt/hibe-1.2.1/lib目录下
	②进入/opt/hive-1.2.1/conf,复制hive-default-xml.template重命名为hive-site.xml
		cp hive-default-xml.template hive-site.xml
	③修改hive-site.xml，
		①vi hive-site.xml
		②非编辑模式删除所有的<property>，:开始行号 结束行号d
		③增加以下mysql配置
			<!--URL-->
            <property>
                <name>javax.jdo.option.ConnectionURL</name>
                <value>jdbc:mysql://hadoop02:3306/hive_metastore?createDatabaseIfNotExist=true&amp;serverTimezone=GMT%2B8&amp;characterEncoding=utf8</value>
                <description>JDBC connect string for a JDBC metastore</description>
              </property>

            <!--username-->
              <property>
                <name>javax.jdo.option.ConnectionUserName</name>
                <value>root</value>
                <description>Username to use against metastore database</description>
              </property>

            <!--password-->
             <property>
                <name>javax.jdo.option.ConnectionPassword</name>
                <value>alex123</value>
                <description>password to use against metastore database</description>
              </property>

            <!--drivername:mysql8的驱动与mysql5不同-->
            <property>
                <name>javax.jdo.option.ConnectionDriverName</name>
                <value>com.mysql.jdbc.Driver</value>
                <description>Driver class name for a JDBC metastore</description>
              </property>
		注意：xml中&是特殊字符，需要转义为实体&amp;
	③配置Mysql服务端
		①新增用于远程连接的用户：create user alex identified by 'alex123'
        	授权：
        		①grant all privileges on *.* to 'alex'@'%'
        		②flush privileges;
	④启动hive服务
		启动hive服务有两种方式：
			①使用hive命令（本地服务）
			②使用hiveserver2（远程服务模式）
				①可以使用hive-jdbc进行连接（idea工程big-data-03-hive已经实现）
				②也可以使用beeline客户端连接
	⑤查看hive数据库的元数据
		使用mysql客户端连接，在navicat mysql客户端使用上述的账号密码登录服务端，观察新建了数据库hive_metastore，数据库中包含hive数据库的元数据表：
            ①DBS，记录数据库的元数据
            ②TBLS，记录表的元数据
            ③COLUMNS_V2,记录字段的元数据
            ④PARTITIONS,PARTITION_PARAMS,记录分区的元数据
            ⑤BUCKETING_COLS，记录分桶的元数据
            ⑥IDXS,INDEX_PARAMS，记录索引的元数据
         注意：hive不支持外键
```

远程服务模式下存在的问题

```
①使用hive-jdbc连接远程的hive服务，存在很多限制，DML不支持删改，会直接报错
②支持DDL和增查，前提是保证hdfs:///usr/alex/hive有rwx权限，hadoop fs -chmod -R 777 /usr
```

### HIVE索引

```
hive对索引的支持比较有限，索引以索引表的形式存放在hive中
【语法】
第一步：创建空索引
create index 索引名 on table 表名(索引字段)
as 'bitmap'  //指定索引处理器
with deferred rebuild //开始时是一个空索引

第二步：创建索引数据
	alter index 索引名 on table 表名 rebuild  //普通表的创建方式
	alter index 索引名 on table 表名 partition(分区字段="分区名") rebuild  //分区表的创建方式

```

### HIVE四种排序

```
①order by,在reduce阶段进行一次全局排序（效率低，生产中禁止使用）
②sort by，在单独的reduce中进行局部排序，不能保证全局有序，一般和distribute by配合使用（用的最多）
	select * from 表名 distribute by 字段1 sort by 字段2 asc/desc
	先按照字段1进行分区（多个reduce），在每个reduce中按照字段2进行排序
③distribute by 【字段】，控制map中的输出在reduce是如何进行划分的，相同的【字段】的记录被划分到同一个reduce
④cluster by 【字段1】，相当于 distribute by 【字段1】 sort by 【字段1】 asc
	只能升序，不够灵活
```

### HIVE分区与分桶

```
分区：
	①类似hdfs不同的目录，对表进行分区相当于按照分区字段将数据放到多个目录中
	
	分区又可以分为静态分区和动态分区：
        静态分区：在insert时指定要插入的分区，生产中分区可能比较多，静态分区不适用
		【语法】 insert into  table 表1 partition(分区字段='分区名') select * from 表2
        动态分区：在insert数据时不指定具体的分区，而由hive执行器根据分区字段的值，插入到相应的分区中
        步骤：
        	①必须，开启动态分区功能（默认true）
        		set hive.exec.dynamic.partition=true
        	②必须，设置非严格模式（动态分区的模式，默认 strict，表示必须指定至少一个分区为静态分区，nonstrict 模式表示允许所有的分区字段都可以使用动态分区）
        		set hive.exec.dynamic.partition.mode=nonstrict
        	③非必须，指定MR最大支持的动态分区数目（默认100，根据生产实际情况指定）
        		set hive.exec.max.dynamic.partitions=1000
        	④非必须，指定MR最大支持创建多少个HDFS文件
        		set hive.exec.max.created.files=10000
        	⑤非必须，当有空分区(分区字段为null)生成时，是否抛出异常，默认false
        	 	set hive.error.on.empty.partition=false
        【语法】insert into table 表1 partition(分区字段) select * from 表2 

	
分桶：
	①类似hdfs同一目录下不同的文件，对表进行分桶相当于按照【分桶字段的值】% 记录条数，将数据放到不同的文件
		与MR的分区很相似
	②使用：
		①为当前会话开启分桶功能:
			set hive.enforce.bucketing = true;//开启分桶
			set mapreduce.job.reduces = -1; //按照分桶数据生成响应数量的reduce（默认只有一个）
		②创建分桶表
			create table (字段 类型...) clustered by (分桶字段 类型) into n buckets
```

### HIVE Snappy压缩

Hive压缩实际是Map阶段和Reduce阶段进行的

##### 开启Map阶段压缩（可以减少Map阶段写入环形缓冲区的数据量）

```
1.开启hive中间传输数据压缩功能
	set hive.exec.compress.intermediate=true;
2.开启Map阶段输出压缩功能：
	set mapreduce.map.output.compress=true;
3.设置Map阶段输出数据的压缩方式为Snappy
	set mapreduce.map.output.compress.codec=org.apache.hadoop.io.compress.SnappyCodec;
4.执行查询语句
```

##### 开启Reduce阶段压缩（减少写回到hdfs的表数据）

```
1.开启hive最终输出数据压缩功能
	set hive.exec.compress.output=true;
2.开启Reduce阶段输出压缩功能
	set mapreduce.output.fileoutputformat.compress=true;
3.设置mapreduce最终数据输出压缩方式
	set mapreduce.output.fileoutputformat.compress.codesc=org.apache.hadoop.io.compress.SnappyCodec;
4.设置mapreduce最终数据输出压缩为块压缩
	set mapreduce.output.fileoutputformat.compress.type=BLOCK;
5.测试输出结果是否为压缩文件
	insert overwrite local directory '/opt/hive-1.2.1/mydb/output/dept-result' select * from dept distribute by position sort by salary;
```

注意，要使用编译了Snappy或者lzo的源码的hadoop版本，才能使用这两种压缩，通过命令查看支持哪些压缩：

​	hadoop checknative

### HIVE文件存储格式

```
hive数据存储有两种方式：
	①行存储
		包含两种数据存储格式：TextFile（默认）、SequenceFile
	②列存储
		包括两种存储格式：Orc、Parquet
##########################################################################
格式				加载方式			加载速度			压缩比			查询效率
									由高到低4-1			由高到低4-1     由高到低5-1		  
##########################################################################
TextFile		load / insert			 4					1			1
SequenceFile  		insert				 3					2			2
Orc					insert				 1					4			4
Parquet				insert               2					3			3
##########################################################################
注意：除了TextFile可以直接将文件文件加载到表中，其它三种格式只能使用insert select方式加载，并且要先将数据以TextFile方式导入临时表temp，再从temp以SequenceFile/Orc/Parquet三种方式导入
```

### 内部表与外部表互转

```
内部表转外部表：ALTER TABLE 表名 SET TBLPROPERTIES("EXTERNAL"="TRUE")
外部表转内部表：ALTER TABLE 表名 SET TBLPROPERTIES("EXTERNAL"="FALSE")
```

### HIVE使用TEZ引擎

```
Tez是Apache最新的支持DAG作业的开源计算框架，它可以将多个有依赖的作业转换为一个作业从而大幅提升DAG作业的性能。Tez可以提供接近实时的查询，性能优于MR。一般用于数据量不大的测试环境
```

##### 配置过程

```
步骤一：下载安装包
	tez官网tez.apache.org下载0.9.1版本的安装包apache-tez-0.9.1-bin.tar.gz,上传到hadoop02(192.168.8.105)的/opt目录，解压后从命名为/opt/tez-0.9.1
步骤二：配置tez
	进入/opt/tez-0.9.1/conf，复制tez-default-template.xml 到 /opt/hive-1.2.1/conf下，重命名为tez-site.xml，删除所有的<property>后添加以下属性：
		  <!-- 将tez的所有jar上传到hdfs -->
          <property>
            <name>tez.lib.uris</name>
            <value>${fs.defaultFS}/tez/tez-0.9.1,${fs.defaultFS}/tez/tez-0.9.1/lib</value>
          </property>
          <!-- 配置tez lib指向hdfs下的路径 -->
          <property>
            <name>tez.lib.uris.classpath</name>
            <value>${fs.defaultFS}/tez/tez-0.9.1,${fs.defaultFS}/tez/tez-0.9.1/lib</value>
          </property>
          <property>
               <name>tez.use.cluster.hadoop-libs</name>
               <value>true</value>
          </property>
          <!-- 配置tez的历史服务 -->
          <property>
               <name>tez.history.logging.service.class</name>  
               <value>org.apache.tez.dag.history.logging.ats.ATSHistoryLoggingService</value>
          </property>
步骤三：hive关联tez
    ①配置/opt/hive-1.2.1/conf/hive-env.sh，增加：
        export TEZ_HOME=/opt/tez-0.9.1    #tez的解压目录
        export TEZ_JARS=""
        for jar in `ls $TEZ_HOME |grep jar`; do
        export TEZ_JARS=$TEZ_JARS:$TEZ_HOME/$jar
        done
        for jar in `ls $TEZ_HOME/lib`; do
        export TEZ_JARS=$TEZ_JARS:$TEZ_HOME/lib/$jar
        done

    export HIVE_AUX_JARS_PATH=/opt/hadoop-2.7.2/share/hadoop/common/lib/hadoop-lzo-0.4.20.jar$TEZ_JARS
	②配置/opt/hive-1.2.1/conf/hive-site.xml,增加：
		<!--更改hive的计算引擎,默认是mr-->
        <property>
            <name>hive.execution.engine</name>
            <value>tez</value>
        </property>
步骤四：上传tez到hdfs集群
	为了使tez直接运行在hdfs集群，需要将tez的安装目录上传到hdfs文件系统/tez下
	①在hadoop02:9000下新建/tez目录
		hadoop fs -mkdir /tez
	②上传目录
		hadoop fs -put /opt/tez-0.9.1 /tez
	
	
```

### 中文乱码问题

```
步骤一：修改mysql编码
	init_connect='SET collation_connection = utf8_unicode_ci' 

    init_connect='SET NAMES utf8'

    character-set-server=utf8  

    collation-server=utf8_unicode_ci

    skip-character-set-client-handshake

	重启mysql服务，使配置生效，service mysql restart
步骤二：修改hive元数据编码
	①登录mysql客户端
	②连接hive元数据库
	③针对不同字段的乱码问题，修改编码
        ①修改表字段和表注解
    alter table COLUMNS_V2 modify column COMMENT varchar(256) character set utf8;
    alter table TABLE_PARAMS modify column PARAM_VALUE varchar(4000) character set utf8;

        ②修改分区字段注解
    alter table PARTITION_PARAMS modify column PARAM_VALUE varchar(4000) character set utf8;
    alter table PARTITION_KEYS modify column PKEY_COMMENT varchar(4000) character set utf8;

        ③修改索引注解
    alter table INDEX_PARAMS modify column PARAM_VALUE varchar(4000) character set utf8;
```

### 案例：数仓项目

```
数仓项目中，需要批量建表及导数据，需要编写shell脚本减少运维工作量。
```

以下是一个大脚本dwd_event_table.sh，dwd层事件表建表及导入数据

```
#!/bin/bash
#dwd层事件表建表/加载数据脚本

if [ -n "$1" ];then
	load_date=$1
else
	load_date=`date -d "-1 day" +%F`
fi

sqls="
use gmall;

CREATE EXTERNAL TABLE IF NOT EXISTS dwd_display_log(
mid_id string,
user_id string,
version_code string,
version_name string,
lang string,
source string,
os string,
area string,
model string,
brand string,
sdk_version string,
gmail string,
height_width string,
app_time string,
network string,
lng string,
lat string,
action string,
goodsid string,
place string,
extend1 string,
category string,
server_time string
)
PARTITIONED BY (dt string)
location '/usr/project_data_warehouse/gmall/dwd/dwd_display_log/';

insert into table dwd_display_log
PARTITION (dt='$load_date')
select 
mid_id,
user_id,
version_code,
version_name,
lang,
source,
os,
area,
model,
brand,
sdk_version,
gmail,
height_width,
app_time,
network,
lng,
lat,
get_json_object(event_json,'$.kv.action') action,
get_json_object(event_json,'$.kv.goodsid') goodsid,
get_json_object(event_json,'$.kv.place') place,
get_json_object(event_json,'$.kv.extend1') extend1,
get_json_object(event_json,'$.kv.category') category,
server_time
from dwd_base_event_log 
where dt='$load_date' and event_name='display';

CREATE EXTERNAL TABLE IF NOT EXISTS dwd_newsdetail_log(
mid_id string,
user_id string, 
version_code string, 
version_name string, 
lang string, 
source string, 
os string, 
area string, 
model string,
brand string, 
sdk_version string, 
gmail string, 
height_width string, 
app_time string,  
network string, 
lng string, 
lat string, 
entry string,
action string,
goodsid string,
showtype string,
news_staytime string,
loading_time string,
type1 string,
category string,
server_time string)
PARTITIONED BY (dt string)
location '/usr/project_data_warehouse/gmall/dwd/dwd_newsdetail_log/';

insert into table dwd_newsdetail_log
PARTITION (dt='$load_date')
select 
mid_id,
user_id,
version_code,
version_name,
lang,
source,
os,
area,
model,
brand,
sdk_version,
gmail,
height_width,
app_time,
network,
lng,
lat,
get_json_object(event_json,'$.kv.entry') entry,
get_json_object(event_json,'$.kv.action') action,
get_json_object(event_json,'$.kv.goodsid') goodsid,
get_json_object(event_json,'$.kv.showtype') showtype,
get_json_object(event_json,'$.kv.news_staytime') news_staytime,
get_json_object(event_json,'$.kv.loading_time') loading_time,
get_json_object(event_json,'$.kv.type1') type1,
get_json_object(event_json,'$.kv.category') category,
server_time
from dwd_base_event_log
where dt='$load_date' and event_name='newsdetail';

CREATE EXTERNAL TABLE IF NOT EXISTS dwd_loading_log(
mid_id string,
user_id string, 
version_code string, 
version_name string, 
lang string, 
source string, 
os string, 
area string, 
model string,
brand string, 
sdk_version string, 
gmail string,
height_width string,  
app_time string,
network string, 
lng string, 
lat string, 
action string,
loading_time string,
loading_way string,
extend1 string,
extend2 string,
type string,
type1 string,
server_time string)
PARTITIONED BY (dt string)
location '/usr/project_data_warehouse/gmall/dwd/dwd_loading_log/';

insert into table dwd_loading_log
PARTITION (dt='$load_date')
select 
mid_id,
user_id,
version_code,
version_name,
lang,
source,
os,
area,
model,
brand,
sdk_version,
gmail,
height_width,
app_time,
network,
lng,
lat,
get_json_object(event_json,'$.kv.action') action,
get_json_object(event_json,'$.kv.loading_time') loading_time,
get_json_object(event_json,'$.kv.loading_way') loading_way,
get_json_object(event_json,'$.kv.extend1') extend1,
get_json_object(event_json,'$.kv.extend2') extend2,
get_json_object(event_json,'$.kv.type') type,
get_json_object(event_json,'$.kv.type1') type1,
server_time
from dwd_base_event_log
where dt='$load_date' and event_name='loading';

CREATE EXTERNAL TABLE IF NOT EXISTS dwd_ad_log(
mid_id string,
user_id string, 
version_code string, 
version_name string, 
lang string, 
source string, 
os string, 
area string, 
model string,
brand string, 
sdk_version string, 
gmail string, 
height_width string,  
app_time string,
network string, 
lng string, 
lat string, 
entry string,
action string,
content string,
detail string,
ad_source string,
behavior string,
newstype string,
show_style string,
server_time string)
PARTITIONED BY (dt string)
location '/usr/project_data_warehouse/gmall/dwd/dwd_ad_log/';

insert into table dwd_ad_log
PARTITION (dt='$load_date')
select 
mid_id,
user_id,
version_code,
version_name,
lang,
source,
os,
area,
model,
brand,
sdk_version,
gmail,
height_width,
app_time,
network,
lng,
lat,
get_json_object(event_json,'$.kv.entry') entry,
get_json_object(event_json,'$.kv.action') action,
get_json_object(event_json,'$.kv.content') content,
get_json_object(event_json,'$.kv.detail') detail,
get_json_object(event_json,'$.kv.source') ad_source,
get_json_object(event_json,'$.kv.behavior') behavior,
get_json_object(event_json,'$.kv.newstype') newstype,
get_json_object(event_json,'$.kv.show_style') show_style,
server_time
from dwd_base_event_log 
where dt='$load_date' and event_name='ad';

CREATE EXTERNAL TABLE IF NOT EXISTS dwd_notification_log(
mid_id string,
user_id string, 
version_code string, 
version_name string, 
lang string,
source string, 
os string, 
area string, 
model string,
brand string, 
sdk_version string, 
gmail string, 
height_width string,  
app_time string,
network string, 
lng string, 
lat string, 
action string,
noti_type string,
ap_time string,
content string,
server_time string
)
PARTITIONED BY (dt string)
location '/usr/project_data_warehouse/gmall/dwd/dwd_notification_log/';

insert into table dwd_notification_log
PARTITION (dt='$load_date')
select 
mid_id,
user_id,
version_code,
version_name,
lang,
source,
os,
area,
model,
brand,
sdk_version,
gmail,
height_width,
app_time,
network,
lng,
lat,
get_json_object(event_json,'$.kv.action') action,
get_json_object(event_json,'$.kv.noti_type') noti_type,
get_json_object(event_json,'$.kv.ap_time') ap_time,
get_json_object(event_json,'$.kv.content') content,
server_time
from dwd_base_event_log
where dt='$load_date' and event_name='notification';

CREATE EXTERNAL TABLE IF NOT EXISTS dwd_active_foreground_log(
mid_id string,
user_id string,
version_code string,
version_name string,
lang string,
source string,
os string,
area string,
model string,
brand string,
sdk_version string,
gmail string,
height_width string,
app_time string,
network string,
lng string,
lat string,
push_id string,
access string,
server_time string)
PARTITIONED BY (dt string)
location '/usr/project_data_warehouse/gmall/dwd/dwd_foreground_log/';

insert into table dwd_active_foreground_log
PARTITION (dt='$load_date')
select 
mid_id,
user_id,
version_code,
version_name,
lang,
source,
os,
area,
model,
brand,
sdk_version,
gmail,
height_width,
app_time,
network,
lng,
lat,
get_json_object(event_json,'$.kv.push_id') push_id,
get_json_object(event_json,'$.kv.access') access,
server_time
from dwd_base_event_log
where dt='$load_date' and event_name='active_foreground';

CREATE EXTERNAL TABLE IF NOT EXISTS dwd_active_background_log(
mid_id string,
user_id string,
version_code string,
version_name string,
lang string,
source string,
os string,
area string,
model string,
brand string,
sdk_version string,
gmail string,
 height_width string,
app_time string,
network string,
lng string,
lat string,
active_source string,
server_time string
)
PARTITIONED BY (dt string)
location '/usr/project_data_warehouse/gmall/dwd/dwd_background_log/';

insert into table dwd_active_background_log
PARTITION (dt='$load_date')
select 
mid_id,
user_id,
version_code,
version_name,
lang,
source,
os,
area,
model,
brand,
sdk_version,
gmail,
height_width,
app_time,
network,
lng,
lat,
get_json_object(event_json,'$.kv.active_source') active_source,
server_time
from dwd_base_event_log
where dt='$load_date' and event_name='active_background';

CREATE EXTERNAL TABLE IF NOT EXISTS dwd_comment_log(
mid_id string,
user_id string,
version_code string,
version_name string,
lang string,
source string,
os string,
area string,
model string,
brand string,
sdk_version string,
gmail string,
height_width string,
app_time string,
network string,
lng string,
lat string,
comment_id int,
userid int,
p_comment_id int, 
content string,
addtime string,
other_id int,
praise_count int,
reply_count int,
server_time string
)
PARTITIONED BY (dt string)
location '/usr/project_data_warehouse/gmall/dwd/dwd_comment_log/';

insert into table dwd_comment_log
PARTITION (dt='$load_date')
select 
mid_id,
user_id,
version_code,
version_name,
lang,
source,
os,
area,
model,
brand,
sdk_version,
gmail,
height_width,
app_time,
network,
lng,
lat,
get_json_object(event_json,'$.kv.comment_id') comment_id,
get_json_object(event_json,'$.kv.userid') userid,
get_json_object(event_json,'$.kv.p_comment_id') p_comment_id,
get_json_object(event_json,'$.kv.content') content,
get_json_object(event_json,'$.kv.addtime') addtime,
get_json_object(event_json,'$.kv.other_id') other_id,
get_json_object(event_json,'$.kv.praise_count') praise_count,
get_json_object(event_json,'$.kv.reply_count') reply_count,
server_time
from dwd_base_event_log
where dt='$load_date' and event_name='comment';

CREATE EXTERNAL TABLE IF NOT EXISTS dwd_favorites_log(
mid_id string,
user_id string, 
version_code string, 
version_name string, 
lang string, 
source string, 
os string, 
area string, 
model string,
brand string, 
sdk_version string, 
gmail string, 
height_width string,  
app_time string,
network string, 
lng string, 
lat string, 
id int, 
course_id int, 
userid int,
add_time string,
server_time string
)
PARTITIONED BY (dt string)
location '/usr/project_data_warehouse/gmall/dwd/dwd_favorites_log/';

insert into table dwd_favorites_log
PARTITION (dt='$load_date')
select 
mid_id,
user_id,
version_code,
version_name,
lang,
source,
os,
area,
model,
brand,
sdk_version,
gmail,
height_width,
app_time,
network,
lng,
lat,
get_json_object(event_json,'$.kv.id') id,
get_json_object(event_json,'$.kv.course_id') course_id,
get_json_object(event_json,'$.kv.userid') userid,
get_json_object(event_json,'$.kv.add_time') add_time,
server_time
from dwd_base_event_log 
where dt='$load_date' and event_name='favorites';


CREATE EXTERNAL TABLE IF NOT EXISTS dwd_praise_log(
mid_id string,
user_id string, 
version_code string, 
version_name string, 
lang string, 
source string, 
os string, 
area string, 
model string,
brand string, 
sdk_version string, 
gmail string, 
height_width string,  
app_time string,
network string, 
lng string, 
lat string, 
id string, 
userid string, 
target_id string,
type string,
add_time string,
server_time string
)
PARTITIONED BY (dt string)
location '/usr/project_data_warehouse/gmall/dwd/dwd_praise_log/';

insert into table dwd_praise_log
PARTITION (dt='$load_date')
select 
mid_id,
user_id,
version_code,
version_name,
lang,
source,
os,
area,
model,
brand,
sdk_version,
gmail,
height_width,
app_time,
network,
lng,
lat,
get_json_object(event_json,'$.kv.id') id,
get_json_object(event_json,'$.kv.userid') userid,
get_json_object(event_json,'$.kv.target_id') target_id,
get_json_object(event_json,'$.kv.type') type,
get_json_object(event_json,'$.kv.add_time') add_time,
server_time
from dwd_base_event_log
where dt='$load_date' and event_name='praise';

CREATE EXTERNAL TABLE IF NOT EXISTS dwd_error_log(
mid_id string,
user_id string, 
version_code string, 
version_name string, 
lang string, 
source string, 
os string, 
area string, 
model string,
brand string, 
sdk_version string, 
gmail string, 
height_width string,  
app_time string,
network string, 
lng string, 
lat string, 
errorBrief string, 
errorDetail string, 
server_time string)
PARTITIONED BY (dt string)
location '/usr/project_data_warehouse/gmall/dwd/dwd_error_log/';

insert into table dwd_error_log
PARTITION (dt='$load_date')
select 
mid_id,
user_id,
version_code,
version_name,
lang,
source,
os,
area,
model,
brand,
sdk_version,
gmail,
height_width,
app_time,
network,
lng,
lat,
get_json_object(event_json,'$.kv.errorBrief') errorBrief,
get_json_object(event_json,'$.kv.errorDetail') errorDetail,
server_time
from dwd_base_event_log 
where dt='$load_date' and event_name='error';
"
$HIVE_HOME/bin/hive -e "$sqls"
```

### HIVE企业调优

##### Fetch抓取（概念：某些情况不使用MapReduce计算）

```
对应参数：hive.fetch.task.conversion
```

| 参数         | 功能                                    |
| ------------ | --------------------------------------- |
| none         | 所有的hql都走MR                         |
| minimal      | 除简单select外，走MR                    |
| more（默认） | 除全局查找，字段查找、limit查找外，走MR |

##### 本地模式（测试环境使用）

```
hive运行在本地模式，可以减少启动job的时间
默认关闭，开启本地模式：
	set hive.exec.mode.local.auto=true;
并且只有同时满足以下两个条件时本地模式有效：
①hive.exec.mode.local.auto.inputbytes.max=234217728，数据集小于128M时
②hive.exec.mode.local.auto.input.files.max=4,最大输入文件个数小于4
```

##### 非严格模式

```
hive提供了一个严格模式，可以防止用户执行那些可能产生意想不到的不好的效果的查询。即某些查询在严格
模式下无法执行。通过设置hive.mapred.mode的值为strict，可以禁止以下3种类型的查询。

①分区表的查询
	必须显示的where中指定分区过滤条件，否则不允许执行
	
②带有order by的查询
	必须配合limit使用，否则不允许执行
	
③限制笛卡尔积的查询
	select * from 表1,表2	不允许执行
	可以用关联表替换，select * from 表1 join 表2 on 表1字段1=表2字段2
	
非严格模式下，没有以上的限制，但是生产中严禁开启非严格模式（很危险，如果数据量巨大，容易导致服务器挂掉），一般在调试环境使用：
	在hive客户端，切换为非严格模式，使用命令 set hive.mapred.mode=nonstrict;
```

##### 开启Map Join

```
Hive中的Join可分为Common Join（Reduce阶段完成join）和Map Join（Map阶段完成join）。

对比Common Join和Map Join
①Common Join,Common Join整个过程包含Map、Shuffle、Reduce阶段。
    ①Map阶段
        读取源表的数据，Map输出时候以Join on条件中的列为key，如果Join有多个关联键，则以这些关联键的组合作为key;
        Map输出的value为join之后所关心的(select或者where中需要用到的)列；同时在value中还会包含表的Tag信息，用于标明此value对应哪个表；
        按照key进行排序

    ②Shuffle阶段，根据key的值进行hash,并将key/value按照hash值推送至不同的reduce中，这样确保两个表中相同的key位于同一个reduce中

    ③Reduce阶段，根据key的值完成join操作，期间通过Tag来识别不同表中的数据。
②Map Join
	①由一个本地Task将小表存放到内存中的HashTable数据结构中
	②之后，由启动一个没有Reduce的MR处理大表，将大表的每一条记录与小表的HashTable关联
	③在Map阶段直接将关联的结果输出到文件
	④由于小表在内存中处理，且没有Reduce阶段，减少了IO的次数，效率比Common Join高

生产中，开启Map Join的过程：
①开启Map Join（默认true）
	set hive.auto.convert.join=true;
②设定小表大小的阈值(默认25M以下是小表)：
	set hive.mapjoin.smalltable.filesize=25000000
```

#### 数据倾斜问题

```
Map阶段相同key的数据会分发给同一个Reduce，当相同key的数据过大时会导致某个Reduce的执行时间过长，这就是数据倾斜问题
```

###### 空key导致数据倾斜

```
如果数据集中存在大量的空字段，且这些空字段又被作为Key的话，会导致这些字段的行数据会进入同一个reducer任务，造成两个问题:
	①数据倾斜，某个reducer产生的part文件过大
	②reducer并发效率低（某个reducer非常忙碌，导致整个job费时严重）
	
解决空KEY的问题：
	①在map阶段，根据实际业务，为空KEY字段赋一个随机的值，使数据能均匀的分发到不同的reducer
	②一般在hql中用if(,,)为空KEY字段赋值
```

###### Group By导致数据倾斜

```
默认情况下，Group By在Reduce阶段完成聚合，如果数据过大，且ReduceTask数量只有一个，会发生数据倾斜
并不是所有的聚合操作都需要在 Reduce 端完成，很多聚合操作都可以先在 Map 端进行
部分聚合，最后在 Reduce 端得出最终结果。

hive中开启Map阶段聚合功能：
	①是否在Map阶段进行聚合（默认true）
		set hive.map.aggr=true
	②Map阶段进行聚合操作的条目数目
		set hive.groupby.mapaggr.checkinterval=100000
	③开启负载均衡（默认false）
		set hive.groupby.skewindata=true
	当选项设定为 true，生成的查询计划会有两个 MR Job。第一个 MR Job 中，Map 的输
出结果会随机分布到 Reduce 中，每个 Reduce 做部分聚合操作，并输出结果，这样处理的结
果是相同的 Group By Key 有可能被分发到不同的 Reduce 中，从而达到负载均衡的目的；第二个 MR Job 再根据预处理的数据结果按照 Group By Key 分布到 Reduce 中（这个过程可以保证相同的 Group By Key 被分布到同一个 Reduce 中），最后完成最终的聚合操作。
```

##### count(distinct)导致数据倾斜

```
count(distinct)操作只能由一个ReduceTask完成，如果数据量过大，会发生数据倾斜。
解决办法：
	先分组再count
```

##### HQL性能优化

```
一、尽量使用索引，避免全表查询
① 在where 和 order by 常用的字段上创建索引，提升效率的核心！但是用不到的字段就别加索引了，反而浪费存储空间；频繁更新的字段也不适合创建索引
② where子句中尽量避免进行nul值判断，少使用!=,<>等操作符，等号左边避免函数、算数和其它表达式运算，这此操作都会导致索引无效，启动全表查询
③ where 子句中尽量少用or，如果一定要多条件并集查询，可以使用Union All替代OR
④ where子句中 LIKE 字符第1个不要是%，这样会启动全表查询

二、多表关联小技巧
①避免笛卡尔积出现，如果多表关联，在符合业务需求情况下，能去重就都去重，比如登录表和充值表就是多条记录表，如果要关联获得登录用户的累计充值金额，则可以将登录表先去重，充值表汇总，再关联
② 多表关联小表在前，大表在后，hive会默认第1个表纳入内存，然后再对大表进行关联计算，小表放前将会大大提升效率
③ 尽量早过滤，减少每个阶段的数据量，过滤条件执行顺序：关联表中的WHERE>ON>WHERE子句，最好先将每个要关联的表都变成子查询语句，先用where 进行过滤，再在ON里面进行过滤，速度不止快10倍！
④ 多表最好都用别名，并且在各子句中都用别名+字段的方式引用，会大大提升效率

三、避免数据倾斜
① 尽量少用COUNT(DISTINCT)，可以用COUNT+GROUP BY，如果一定要用，可以将重复特别多的（例如NULL)先过滤掉，再单独处理
② 选择分布较为均匀的键作为关联KEY，如用户ID，日期，不要用性别，年龄等分布不均的字段
③ 在使用JOIN时，关联键存在大量空值或者某一特殊值，如”NULL”，可以空值单独处理，不参与关联，或者空值或特殊值加随机数作为关联键；不同数据类型的字段关联，则可以转换为同一数据类型之后再做关联

四、其余优化技巧
① 任何时候都要尽量避免使用 SELECT *** ，能用列名就用列名
② 尽量不要建立临时表**，以减少系统表资源的浪费，但如果源表实在太大，必须建临时表时，可以用CREATE TABLE+INSERT TABLE的方式分批插入，会比仅用CREATE+TABLE要快很多
③ 子查询会比join快，虽然并不绝对，但是绝大部分情况下都是
④ 尽量少排序，如果一定要排序可以用sort by 替换order by ,order by 是全局排序，会占用大量的资源，sort by是局部排序，与distribute by一起使用，速度更快，但这个也要看实际需求
⑤ 尽量用Union All替代Union，因为Union是要组合后排序去重的，Union All是直接扫描源表即可，效率更高
⑥ 尽量使用数值型字段而不是字符型字段
```

##### 合理设置MapTask和ReduceTask数量

```
什么时候增加MapTask?
	当 input 的文件都很大，任务逻辑复杂，map 执行非常慢的时候，可以考虑增加 Map
数，来使得每个 map 处理的数据量减少，从而提高任务的执行效率。
如何增加MapTask？
	MapTask的数量 =块大小（默认128M）/ 切片大小 共同决定，默认情况下切片大小等于块大小，只需要改变切片大小就可以改变MapTask数量
	set mapreduce.input.fileinputformat.split.maxsize=100，(单位字节)
	
什么时候增加ReduceTask？
	ReduceTask的数量由分区数决定，如果数据量很大，需要增加ReduceTask数，可以提高并行处理的速度，（但过多的ReduceTask本身也会耗费资源，如果并行很大，会占用集群过多的资源，影响集群稳定性）
如何增加ReduceTask？
	①每个ReduceTask处理的数据量（单位字节，默认256M）
		set hive.exec.reducers.bytes.per.reducer=256000000
	②每个MR最大的ReduceTask数量（默认1009）
		set hive.exec.reducers.max=1009
```

##### 合并小文件

```
SET hive.merge.mapfiles = true; -- 默认true，在map-only任务结束时合并小文件
SET hive.merge.mapredfiles = true; -- 默认false，在map-reduce任务结束时合并小文件
SET hive.merge.size.per.task = 268435456; -- 默认256M
SET hive.merge.smallfiles.avgsize = 16777216; -- 当输出文件的平均大小小于该值时，启动一个独立的map-reduce任务进行文件merge
```

