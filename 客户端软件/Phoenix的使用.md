# Phoenix的使用

### 简介

```
 Phoenix是一个开源的HBase SQL层，为Hbase提供了标准的sql查询操作，支持标准SQL中大部分特性:条件运算,分组，分页，等高级查询语法。
```

### 安装

```
步骤一：
	从官网http://archive.apache.org/dist/phoenix/下载相应版本的gz包，本机使用的apache-phoenix-4.14.1-HBase-1.3-bin.tar.gz
步骤二：
	使用sftp工具上传到VMWare主机hadoop02（192.168.8.105）的/opt目录并解压，重命名解压后的目录名称phoenix-4.14.1-HBase-1.3
步骤三：配置信息
	①进入/opt/phoenix-4.14.1-HBase-1.3/bin,将当前目录配置到系统环境变量中：
        ①编辑/etc/profile文件，增加
                export PHOENIX_HOME=/opt/phoenix-4.14.1-HBase-1.3
                export PATH=$PATH:$PHOENIX_HOME/bin
		②使配置立刻生效,source /etc/profile
	②关联hbase
		①进入/opt/phoenix-4.14.1-HBase-1.3,将phoenix-4.14.2-HBase-1.3-client.jar和phoenix-4.14.2-HBase-1.3-server.jar拷贝到/opt/hbase-1.3.1/lib下
            cp /opt/phoenix-4.14.1-HBase-1.3/phoenix-4.14.2-HBase-1.3-client.jar /opt/hbase-1.3.1/lib/
            cp /opt/phoenix-4.14.1-HBase-1.3/phoenix-4.14.2-HBase-1.3-server.jar /opt/hbase-1.3.1/lib/
		②将依赖的jar分发到hadoop03和hadoop04
			xsync /opt/hbase-1.3.1/lib/phoenix-4.14.2-HBase-1.3-client.jar
			xsync /opt/hbase-1.3.1/lib/phoenix-4.14.2-HBase-1.3-server.jar
步骤四：启动
	①启动zookeeper集群、hadoop集群、hbase集群
		zkServer-batch-start
		start-dfs.sh
		start-hbase.sh
	②启动phoenix客户端
		/opt/phoenix-4.14.1-HBase-1.3/bin/sqlline.py hadoop02,hadoop03,hadoop04:2181
		注意：zookeeper集群地址的写法比较特殊
```

### 基本语法

```
DDL操作：
查看所有表：
	!tables
退出客户端：
	!quit
创建表：（必须指定主键，因为hbase中一定有主键）
	create table "表名"(字段1 属性 primary key，字段2 属性)
	或者
	create table "表名"(字段1 属性,字段2 属性 constraint 主键名 primary key(主键字段名1,主键字段名2))
删除表：
	drop  table   "表名"

DML操作：
插入/修改数据：（不支持多个记录）
	upsert into 表名 values(多个字段值),注意varchar类型用''不用""
	注意：修改的原理：按照相同的主键进行覆盖
删除数据：
	delete from 表名 where condition
```

### 安装可视化工具SQuirrel（中文名松鼠）（类似于mysql的客户端是navicat）

```
步骤一：下载
	在window系统下载Phoenix的可视化客户端工具SQuirrel
步骤二：安装
	在cmd命令行运行:java -jar squirrel-sql-4.0.0-standard.jar
	弹出安装界面，默认安装即可
步骤三：配置
	将apache-phoenix-4.14.1-HBase-1.3-bin.tar.gz包下/lib目录中的phoenix-4.14.2-HBase-1.3-client.jar拷贝到SQuirrel安装目录的/lib中
步骤四：使用
	打开软件 -> 进入界面
	①Drivers -> 配置jdbc驱动连接信息
        Name：数据库连接名
        Example URL填写: jdbc:phoenix:hadoop02,hadoop03,hadoop04:2181
    ②选中数据库连接名，右键单击"Connect"后连接
```

### Phoenix操作hbase

```
默认情况下，phoenix跟hbase中的表是没有映射起来的，phoenix无法直接操作hbase中的表
映射的方式有两种：
假如：
	hbase中存在表stu,列族为cf1，列为cf1:name,cf1:age,cf1:gender
使用客户端工具SQuirrel：
    ①视图映射（phoenix只能读取hbase中的同名表）
        create view "stu"(id bigint primary key,"cf1"."name" varchar,"cf1"."age" varchar,"cf1"."gender" varchar)
        注意：视图映射只能通过视图名查询hbase
    ②表映射（phoenix能读写hbase中的同名表）
    	create table "stu"(id bigint primary key,"cf1"."name" varchar,"cf1"."age" varchar,"cf1"."gender" varchar)
    注意：对映射视图或者映射表进行操作时，一定在视图名和表名上加""
 
视图映射和表映射的对比与总结：
    ①相比于直接创建映射表，视图的查询效率会低，原因是：创建映射表的时候，Phoenix会在表中创建一些空的键值对，这些空键值对的存在可以用来提高查询效率。
    ②使用create table创建的关联表，如果对表进行了修改，源数据也会改变，同时如果关联表被删除，源表也会被删除。但是视图就不会，如果删除视图，源数据不会发生改变。
```

### phoenix对比hbase

|              | phoenix                  | hbase                       |
| ------------ | ------------------------ | --------------------------- |
| 本质         | 为hbase提供sql功能的插件 | 分布式海量存储的NoSQL数据库 |
| 客户端       | SQuirrel                 | hbase shell                 |
| 数据存放路径 | hdfs                     | hdfs                        |
| hdfs表名     | 强制转换为大写的表名     | 不强制                      |

### phoenix索引（实现hbase二级索引）

```
phoenix支持通过索引对hbase表进行检索，需要配置Hbase支持Phoenix创建二级索引：
①配置HBase的HRegionServer节点（主机hadoop02）
	①编辑/opt/hbase-1.3.1/conf/hbase-site.xml，增加：
<property>
    <name>hbase.regionserver.wal.codec</name>
    <value>
    	org.apache.hadoop.hbase.regionserver.wal.IndexedWALEditCodec
    </value>
</property>

<property>
    <name>hbase.region.server.rpc.scheduler.factory.class</name>
    <value>
    	org.apache.hadoop.hbase.ipc.PhoenixRpcSchedulerFactory
    </value>
</property>

<property>
    <name>hbase.rpc.controllerfactory.class</name>
    <value>
    	org.apache.hadoop.hbase.ipc.controller.ServerRpcControllerFactory
	</value>
</property>
②配置HBase的HMaster节点（主机hadoop02,hadoop03,hadoop04）
	①编辑/opt/hbase-1.3.1/conf/hbase-site.xml，增加：
<!-- phoenix master 配置参数 -->
<property>
    <name>hbase.master.loadbalancer.class</name>
    <value>org.apache.phoenix.hbase.index.balancer.IndexLoadBalancer</value>
</property>

<property>
    <name>hbase.coprocessor.master.classes</name>
    <value>org.apache.phoenix.hbase.index.master.IndexMasterObserver</value>
</property>
```

### 解决热点问题-加盐（类似hbase预分区）

```
如果主键的是顺序的序列，在频繁插入phoenix 到表中，最终会数据不停的落在一个regionServer中，容易造成热点问题。所以尽量将id打散,数据会均匀的分布在各个region中。

设置盐值：
create table "表名"(字段1 属性 primary key，字段2 属性) SALT_BUCKETS=3;盐值=3

设置盐值的原则：
	hbase节点数的倍数，最少是1倍
```

