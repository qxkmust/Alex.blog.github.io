# Kettle的使用

### 简介

```
Kettle是一款国外开源的ETL工具，纯java编写，可以在Windows、Linux、Unix上运行，功能包括：数据抽取、质量检测、数据清洗、数据转换、数据过滤。
```

### windows下安装

```
步骤一：
	从官网https://sourceforge.net/projects/pentaho/files/Data%20Integration/下载相应版本的zip包，本机使用的pdi-ce-8.2.0.0-342.zip
步骤二：window下安装
	直接将zip文件解压到D://kettle_8.2-data_integration
步骤三：配置
	①修改解压目录下的D://kettle_8.2-data_integration\plugins\pentaho-big-data-plugin下的plugin.properties，设置active.hadoop.configuration=hdp26
	②从集群中拷贝：
    	hdfs-site.xml
    	core-site.xml
    	mapred-site.xml
    	yarn-site.xml
    	hbase-site.xml
    	hive-site.xml
    覆盖D://kettle_8.2-data_integration\plugins\pentaho-big-data-plugin\hadoop-configurations\hdp26目录下的同名文件
步骤四：运行
	双击Spoon.bat即可运行
```

### linux下安装

```
步骤一：
	从官网https://sourceforge.net/projects/pentaho/files/Data%20Integration/下载相应版本的zip包，本机使用的pdi-ce-8.2.0.0-342.zip
步骤二：安装
	直接将zip文件解压到/opt/kettle_8.2-data_integration
步骤三：配置
	①修改解压目录下的/opt/kettle_8.2-data_integration/plugins/pentaho-big-data-plugin下的plugin.properties，设置active.hadoop.configuration=hdp26
	②从集群中拷贝：
    	hdfs-site.xml
    	core-site.xml
    	mapred-site.xml
    	yarn-site.xml
    	hbase-site.xml
    	hive-site.xml
    覆盖/opt/kettle_8.2-data_integration/plugins/pentaho-big-data-plugin/hadoop-configurations/hdp26目录下的同名文件
步骤四：运行
	通过.sh脚本运行指定的job或者transform
```

### 组件

```
Kettle家族目前包括4个产品：Spoon、Pan、CHEF、Kitchen。

SPOON 允许你通过图形界面来设计ETL转换过程（Transformation）。

PAN 允许你批量运行由Spoon设计的ETL转换 (例如使用一个时间调度器)。Pan是一个后台执行的程序，没有图形界面。

CHEF 允许你创建任务（Job）。 任务通过允许每个转换，任务，脚本等等，更有利于自动化更新数据仓库的复杂工作。任务通过允许每个转换，任务，脚本等等。任务将会被检查，看看是否正确地运行了。

KITCHEN 允许你批量使用由Chef设计的任务 (例如使用一个时间调度器)。KITCHEN也是一个后台运行的程序。
```

### 两种设计方式

![img](file:///C:\Users\Administrator\AppData\Local\Temp\ksohtml87540\wps1.png)

### 案例

#### 案例一：mysql数据同步

```
需求：
	使用“转换”方式，将192.168.8.105:3306/main_db下的tb_stu表数据导入到192.168.8.105:3306/kettle下的tb_stu
解决：
	①在192.168.8.105创建kettle数据库，在kettle下创建tb_stu表：
		create database kettle default character set utf8 collate utf8_general_ci;
		create table tb_stu like main_db.tb_stu;
	②在window下，启动Sqoop.bat程序：
		①创建“表输入”，配置数据库连接，指定输入的表，指定sql语句“select * from tb_stu”
		②"插入/更新"，配置数据库连接，指定输出的表
	③先保存转换到D:\Kettle_8_2_data_integration\MyWorkplace\mysql_to_mysql_ktr.ktr,
	④点击“Run”运行转换，也可以调用Pan.bat脚本命令：
		Pan.bat /file ./MyWorkplace/mysql_to_mysql_ktr.ktr
```

#### 案例二：mysql数据同步

```
需求：
	①使用“作业”方式，将192.168.8.105:3306/main_db下的tb_stu表数据导入到192.168.8.105:3306/kettle下的tb_stu
	②调用sql删除kettle.tb_stu中id最大的记录
解决：
	①在192.168.8.105创建kettle数据库，在kettle下创建tb_stu表：
		create database kettle default character set utf8 collate utf8_general_ci;
		create table tb_stu like main_db.tb_stu;
	②在window下，启动Sqoop.bat程序：
		①新建“作业”，可以配置定时执行
		②创建“转换”，选取mysql_to_mysql_ktr.ktr作为转换
		③创建"Sql"，编写sql：delete from tb_stu where id = 4
	③先保存转换到D:\Kettle_8_2_data_integration\MyWorkplace\mysql_to_mysql_kjb.kjb,
	④点击“Run”运行转换，也可以调用Pan.bat脚本命令：
		Kitchen.bat /file ./MyWorkplace/mysql_to_mysql_kjb.kjb
```

#### 案例三：hive到hdfs

```
需求：将hive中的表emp与表deptl连接后导入到hdfs
实现：
	①创建"转换"
	②创建“表输入”，命名hive_emp,配置hive连接，编写sql：select * from emp
	③创建“表输入”，命名hive_dept,配置hive连接，编写sql:select * from dept
	④在hive_emp后创建“排序记录”，配置以关联字段“deptno”升序（可能是bug，使用其他字段出错）
	⑤在hive_dept后创建“排序记录”，配置以关联字段“deptno”升序
	⑥创建“记录集连接”，配置关联字段和连接方式
	⑦创建“hadoop file output”，指定hdfs路径、输出字段
	⑧先保存转换到D:\Kettle_8_2_data_integration\MyWorkplace\hive_to_hdfs.ktr,
	然后点击“Run”运行转换
```

### kettle与sqoop对比

```
Kettle中有两种脚本文件，transformation和job，transformation完成针对数据的基础转换，job则完成整个工作流的控制。
Sqoop主要用于在Hadoop(Hive)与传统的数据库(mysql、postgresql...)间进行数据的传递

kettle有图形化的操作界面，只需要描述你想做什么，而不是你想怎么做。
sqoop没有图形化界面，具体的数据流向需要手工配置。

kettle底层使用多线程以提高效率
Sqoop专为大数据批量传输设计，能够分割数据集并创建Hadoop任务来处理每个区块。

kettle可以利用transformation在数据传输过程中对数据的一些转换处理
Sqoop只是一个用来将Hadoop和关系型数据库中的数据相互转移的工具

kettle数据的具体流向可以指定，可以是各种数据的存储工具
sqoop只是完成hdfs到关系型数据库 或者 关系型数据库到hdfs的数据传输，在传输的过程中保证传输数据的类型
```

| 对比         | kettle                  | sqoop                            |
| ------------ | ----------------------- | -------------------------------- |
| 数据抽取     | 任意输入和输出          | 关系数据库 与 非关系数据导入导出 |
| 数据过滤     | 界面配置（生成sql）实现 | 手动sql实现                      |
| 数据质量检测 | 界面配置（生成sql）实现 | 手动sql实现                      |
| 图形界面     | Spoon                   | 无                               |
| 中间数据转换 | 支持                    | 不支持                           |
| ETL          | 完全实现                | 部分实现                         |

