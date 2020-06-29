# Sqoop的使用

### 安装配置

```
步骤一：
	从官网http://www.apache.org/dyn/closer.lua/sqoop/下载相应版本的gz包，本机使用的sqoop-1.4.6.bin__hadoop-2.0.4-alpha.tar.gz
步骤二：
	使用sftp工具上传到VMWare主机hadoop02（192.168.8.105）的/opt目录并解压，重命名解压后的目录名称sqoop-1.4.6
步骤三：配置信息
	①进入/opt/sqoop-1.4.6,将当前目录配置到系统环境变量中：
        ①编辑/etc/profile文件，增加
                export SQOOP_HOME=/opt/sqoop-1.4.6
                export PATH=$PATH:$SQOOP_HOME/bin
		②使配置立刻生效,source /etc/profile
	②修改/opt/sqoop-1.4.6/conf/sqoop-env.sh
        #增加hcatalog环境变量，解决导出parquet格式到mysql
    	export HCAT_HOME=/opt/hive-1.2.1/hcatalog
    	
		#Set path to where bin/hadoop is available
        export HADOOP_COMMON_HOME=/opt/hadoop-2.7.2

        #Set path to where hadoop-*-core.jar is available
        export HADOOP_MAPRED_HOME=/opt/hadoop-2.7.2

        #set the path to where bin/hbase is available
        export HBASE_HOME=/opt/hbase-1.3.1

        #Set the path to where bin/hive is available
        export HIVE_HOME=/opt/hive-1.2.1

        #Set the path for where zookeper config dir is
        export ZOOCFGDIR=/opt/zookeeper-3.4.14
	②登录MVN仓库https://mvnrepository.com/下载hadoop02的mysql 5.5.52版本对应的驱动，这里使用mysql-connector-java-5.1.25.jar，放置到/opt/sqoop-1.4.6/lib/下
	③验证sqoop是否可以连接mysql服务，
		bin/sqoop list-databases --connect jdbc:mysql://hadoop02:3306/ --username alex --password alex123
	注意sqoop是客户端工具，不需要后台运行
```

### 常用命令

-  create-hive-table  Import a table definition into Hive

-  export             Export an HDFS directory to a database table

- help               List available commands

- import             Import a table from a database to HDFS

- import-all-tables  Import tables from a database to HDFS
-   import-mainframe   Import datasets from a mainframe server to HDFS
-   list-databases     List available databases on a server
-   list-tables        List available tables in a database
-   version            Display version information

### 案例一：从RDBMS 到HDFS

##### 案例1.1：导入整张表

```
需求分析：
	①在hadoop02的mysql数据库sqoop中创建表staff表
	②将sqoop_source的数据导入到hdfs://hadoop02:9000/sqoop目录下
具体实现：
	①登录mysql客户端，创建数据库、表以及插入数据：
		create database sqoop;
		create table if not exists staff(id int,name varchar(20));
		insert into staff values(1,'aaa'),(2,'bbb'),(3,'ccc'),(4,'ddd');
	②将数据导出到HDFS，使用命令：
bin/sqoop import \										//sqoop命令
--connect jdbc:mysql://hadoop02:3306/sqoop \			//mysql的服务器地址
--username alex \										//用户名
--password alex123 \										//密码
--table staff \											//	表名
--target-dir /sqoop/staff \								//导出的output路径
--delete-target-dir \					    //如果output路径已经存在就先删除，否则再次会报错
--num-mappers 1 \								//指定并行处理的mapper数量，如果mapper大于1，需要使用split-by参数指定依据哪个字段（一般是主键或唯一键）进行切片
--fields-terminated-by "\t"							//指定数据存放的分隔符

```

##### 案例1.2：查询导入

```
需求分析：
	按照查询条件导入指定的数据
具体实现：
	使用命令：
sqoop import \
--connect jdbc:mysql://hadoop02:3306/sqoop \
--username alex \
--password alex123 \
--target-dir /sqoop/staff \
--delete-target-dir \
--num-mappers 1 \
--fields-terminated-by "\t"	\
--query 'select * from staff where id <=3 and $CONDITIONS;'	//查询语句
```

##### 案例1.3：导入指定列

```
需求分析：
	将staff表中的name列导入到HDFS下的/sqoop/staff目录
具体实现：
sqoop import \
--connect jdbc:mysql://hadoop02:3306/sqoop \
--username alex \
--password alex123 \
--target-dir /sqoop/staff \
--delete-target-dir \
--num-mappers 1 \
--fields-terminated-by "\t"	\
--columns name \				//指定列名
--table staff						//指定表名
```

##### 案例1.4：使用sqoop关键字筛选查询导入数据

```
需求分析：
	使用sqoop关键字【--where】筛选查询导入数据
具体实现：
sqoop import \
--connect jdbc:mysql://hadoop02:3306/sqoop \
--username alex \
--password alex123 \
--target-dir /sqoop/staff \
--delete-target-dir \
--num-mappers 1 \
--fields-terminated-by "\t"	\
--table staff \
--where "id in (1,2)"		//查询条件，类似【--query】
```

### 案例二：从RDBMS  到HIVE

##### 案例2.1：导入整张表

```
需求分析：
	将mysql下sqoop数据库中的staff表导入到Hive
具体实现：
	使用命令：
sqoop import \
--connect jdbc:mysql://hadoop02:3306/sqoop \
--username alex \
--password alex123 \
--table staff \
--delete-target-dir \
--num-mappers 1 \
--hive-import \
--fields-terminated-by "\t"	\
--hive-overwrite \
--hive-table sqoop_to_hive_staff

注意：
	原理：第一步先将数据临时导入到hdfs /user/alex/staff目录，第二步将/user/alex/staff目录下的数据迁移到Hive仓库
```

### 案例三：从RDBMS 到HBASE

##### 案例3.1：导入整张表

```
需求分析：
	将mysql下sqoop数据库中的staff表导入到Hbase
具体实现：
	使用命令：
sqoop import \
--connect jdbc:mysql://hadoop02:3306/sqoop \
--username alex \
--password alex123 \
--table staff \
--columns "id,name" \		//指定列
--column-family "cf1" \		//指定列族
--hbase-create-table \	   //如果hbase表不存在就自动创建，不过因为sqoop与hbase的版本不兼容可能不生效
--hbase-row-key "id" \		//指定Row key
--hbase-table "sqoop_to_hbase_staff" \	//表名
--num-mappers 1 \			//mapper并行任务数
--split-by id				//分区字段
```

### 案例四：HIVE/HDFS导出数据到RDBMS 

##### 案例4.1：导出整张表

```
需求分析：
	将Hive数据存放路径/usr/alex/hive/sqoop_to_hive_staff表导入到mysql的staff表
具体实现：
	使用命令：
sqoop export \
--connect jdbc:mysql://hadoop02:3306/sqoop \
--username alex \
--password alex123 \
--table staff \  	//要导出到关系数据库的哪个表，必须已经建好
--num-mappers 1 \
--export-dir /usr/alex/hive/sqoop_to_hive_staff \		//Hive表存放的hdfs路径
--input-fields-terminated-by "\t"	   //跟Hive表中存放时的分割规则有关系
--staging-table  //创建一张临时表，用于存放所有事务的结果，然后将所有事务结果一次性导入到目标表中，防止错误。
```

### 生产中哪些坑

##### 存储NULL

```
由于hive中的NULL在底层是以"\N"存储，而mysql中NULL在底层就是NULL，加上以下参数解决导入时NULL类型存储的问题
--null-string '\\N'    		//string类型的NULL类型用'\N'存储
--null-non-string '\\N'  //非string类型的NULL类型用'\N'存储

同样的，导出时也存在NULL类型存储的问题，需要增加以下参数：
--input-null-string	'\\N' 		//将'\N'解析为NULL类型
--input-null-non-string '\\N' 	//将'\N'解析为NULL类型
```

##### 存储日期

```
由于hive是按照timestamp格式（YYYY-MM-dd hh:mm:ss.ff）存储日期（date，datetime），将mysql中的日期字段导入到hive中，会导致与原来的格式不一致，加上以下参数可以去除.ff
--map-column-hive "RDBMS日期字段1=timestamp,RDBMS日期字段2=timestamp"
```

##### parquet格式无法导出

```
hive中采用parquet 或者orc存储的表导出数据到mysql可能会无法成功，解决办法：
①不能使用--export-dir
②而是使用：
    --hcatalog-database 数据库名 //hive中的数据库名 
    --hcatalog-table 表名 //hive中的表名
```

##### 数据导出一致性

```
sqoop导入mysql时，如果部分mr任务挂掉了，导致数据只有部分导入到了mysql，解决办法：
增加参数：
--staging-table  //创建一张临时表，用于存放所有事务的结果，然后将所有事务结果一次性导入到目标表中，防止错误。
```

