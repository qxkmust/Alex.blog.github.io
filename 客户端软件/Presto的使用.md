# Presto的使用

### 简介

```
Presto是一个facebook开源的分布式SQL查询引擎，适用于交互式分析查询，数据量支持GB到TB字节。
优点：
	多数据源、支持SQL、扩展性（可以自己扩展新的connector）、混合计算（同一种数据源的不同库 or表；将多个数据源的数据进行合并）、高性能、流水线（pipeline）
```

### 架构

![presto架构](D:\Git\MyRepository\Alex.blog.github.io\Images\presto架构.webp)

### 安装Presto服务

```
步骤一：
	从官网https://repo1.maven.org/maven2/com/facebook/presto/presto-server下载相应版本的gz包，本机使用的presto-server-0.196.tar.gz 
步骤二：
	使用sftp工具上传到VMWare主机hadoop02（192.168.8.105）的/opt目录并解压，重命名解压后的目录名称/opt/presto-server-0.196
步骤三：配置信息
	①进入/opt/presto-server-0.196,将当前目录配置到系统环境变量中：
        ①编辑/etc/profile文件，增加
                export PRESTO_HOME=/opt/presto-server-0.196
                export PATH=$PATH:$PRESTO_HOME/bin
		②使配置立刻生效,source /etc/profile
	②新建目录/opt/presto-server-0.196/data，用于存放数据文件
	③新建目录/opt/presto-server-0.196/etc，用于存放配置文件
		进入/opt/presto-server-0.196/etc
		①创建jvm.config文件，添加 ：
			-server
            -Xmx16G
            -XX:+UseG1GC
            -XX:G1HeapRegionSize=32M
            -XX:+UseGCOverheadLimit
            -XX:+ExplicitGCInvokesConcurrent
            -XX:+HeapDumpOnOutOfMemoryError
            -XX:+ExitOnOutOfMemoryError
         ②创建目录/opt/presto-server-0.196/etc/catalog，并在该目录下创建hive.properties文件（用于关联hive）
         	connector.name=hive-hadoop2
			hive.metastore.uri=thrift://hadoop02:9083
	 ④分发到hadoop03和hadoop04
	 	xsync /opt/presto-server-0.196
	 ⑤分发完后，分别在三个主机配置node属性
	 	hadoop02下，/opt/presto-server-0.196/etc目录，创建node.properties,添加：
	 		node.environment=production
            node.id=ffffffff-ffff-ffff-ffff-ffffffffffff
            node.data-dir=/opt/presto-server-0.196/data
	 	hadoop03下，/opt/presto-server-0.196/etc目录，创建node.properties,添加：
	 		node.environment=production
            node.id=ffffffff-ffff-ffff-ffff-fffffffffffe
            node.data-dir=/opt/presto-server-0.196/data
        hadoop04下，/opt/presto-server-0.196/etc目录，创建node.properties,添加：
	 		node.environment=production
            node.id=ffffffff-ffff-ffff-ffff-fffffffffffd
            node.data-dir=/opt/presto-server-0.196/data
        注意：保证node id不能相同
     ⑥配置一主（coordinator）二从（worker）
     	hadoop02下，/opt/presto-server-0.196/etc目录，创建config.properties,添加：
     		coordinator=true
            node-scheduler.include-coordinator=false
            http-server.http.port=8881
            query.max-memory=50GB
            discovery-server.enabled=true
            discovery.uri=http://hadoop02:8881
        hadoop03下，/opt/presto-server-0.196/etc目录，创建config.properties,添加：
     		coordinator=false
            http-server.http.port=8881
            query.max-memory=50GB
            discovery.uri=http://hadoop02:8881
        hadoop04下，/opt/presto-server-0.196/etc目录，创建config.properties,添加：
     		coordinator=false
            http-server.http.port=8881
            query.max-memory=50GB
            discovery.uri=http://hadoop02:8881
步骤四：启动
	①presto依赖hive服务，先启动hive服务
		nohup hive --service metastore >/dev/null 2>&1 &
	②分别在hadoop02、hadoop03、hadoop04启动presto服务
		前台启动，bin/launcher run
		或
		后台启动，bin/launcher start
步骤五：验证
	后台服务启动后，日志会记录到/opt/presto-server-0.196/data/var/log/server.log
	访问http://hadoop02:8881，查看presto监控页面
```

### 安装Presto可视化客户端工具

```
yanagishima是一个开源的可视化客户端，可用于Presto、Hive、Elasticsearch、Spark

在主机hadoop02安装:
步骤一：编译源码
	yanagishima是开源项目，源码在github，需要下载源码手动编译，教程见官网：https://github.com/yanagishima/yanagishima，最终编译成yanagishima-18.0.zip
步骤二：安装
	①如果没有unzip工具，需要先安装：
		①在http://www.rpmfind.net/linux/rpm2html/search.php下载对应Centos7 64bit版本的unzip-6.0-21.el7.x86_64.rpm
		②rpm -ivh unzip-6.0-21.el7.x86_64.rpm，完成安装
	②将编译后的压缩文件yanagishima-18.0.zip，上传到hadoop02 /opt目录，使用命令解压到/opt/yanagishima-18.0目录：
		unzip yanagishima-18.0.zip
步骤三：配置
	进入/opt/yanagishima-18.0/conf目录，编辑yanagishima.properties，添加：
		jetty.port=7080
        presto.datasources=mypresto			//数据源名称，可以任意
        presto.coordinator.server.mypresto=http://hadoop02:8881 //presto主节点地址
        catalog.mypresto=hive
        schema.mypresto=default
        sql.query.engines=presto
    进入
步骤四：启动
	方式一：进入/opt/yanagishima-18.0，使用命令：
			nohup bin/yanagishima-start.sh >./yanagishima.log 2>&1 &
		注意：必须在根目录下启动，否则报错
	方式二：使用ssh命令以交互模式启动伪终端
		ssh -t hadoop02 "cd /opt/yanagishima-18.0;source /etc/profile;./bin/yanagishima-start.sh;exec $SHELL"
		【命令】ssh -t【开启伪终端，迫使ssh客户端以交互模式工作】 "多个shell命令;bash【指定shell执行器是bash】"
步骤五：验证
	访问http://hadoop02:7080
```

### 对比HIVE

|              | Presto           | Hive                           |
| ------------ | ---------------- | ------------------------------ |
| 执行速度     | 基于内存，速度快 | 基于MR，数据从磁盘读写，速度慢 |
| 大数据量运算 | 适合GB到TB       | 适合海量级别PB                 |

![计算框架对比图](D:\Git\MyRepository\Alex.blog.github.io\Images\计算框架对比图.png)