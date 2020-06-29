# AZKABAN的使用

### 简介

```
azkaban一个批量工作流任务调度器，用于在一个工作流内以一个特定的顺序运行一组工作和流程。Azkaban使用job配置文件建立任务之间的依赖关系，并提供一个易于使用的web用户界面维护和跟踪你的工作流。
```

### 安装

```
在主机hadoop02上安装：
①azkaban是开源项目，源码在github，需要下载源码手动编译，教程见官网：		
	https://github.com/azkaban/azkaban
	三个工程，编译①②：
	①azkaban-exec-server工程，需要编译成.tar.gz文件
	②azkaban-web-server工程，需要编译成.tar.gz文件
	③azkaban-db工程，是mysql的db表，不需要编译，需要导其中的sql
②这里使用编译之后的版本2.5.0,安装包见github：
分别是：	
	azkaban-web服务：azkaban-web-server-2.5.0.tar.gz
	azkaban-executor服务：azkaban-executor-server-2.5.0.tar.gz
	azkaban-sql包：azkaban-sql-script-2.5.0.tar.gz
③解压
	创建/opt/azkaban目录，
	①将azkaban-web-server-2.5.0.tar.gz解压到/opt/azkaban目录，重命名azkaban-web-2.5.0
	②将azkaban-executor-server-2.5.0.tar.gz解压到/opt/azkaban目录，重命名azkaban-executor-2.5.0
	③将azkaban-sql-script-2.5.0.tar.gz解压到/opt/azkaban目录，重命名azkaban-sql-2.5.0
④配置环境变量
	①在/etc/profile添加：
		#配置azkaban-executor环境变量
        export AZKABAN_EXECUTOR_HOME=/opt/azkaban/azkaban-executor-2.5.0
        export PATH=$PATH:$AZKABAN_EXECUTOR_HOME/bin

        #配置azkaban-web环境变量
        export AZKABAN_WEB_HOME=/opt/azkaban/azkaban-web-2.5.0
        export PATH=$PATH:$AZKABAN_WEB_HOME/bin
	②立刻生效，source /etc/profile
⑤新建azkaban数据库
	①使用远程账号登录mysql客户端
	②创建数据库，起名azkaban
		create database azkaban
	③导表
		source /opt/azkaban/azkaban-sql-2.5.0/create-all-sql-2.5.0.sql
⑥生成秘钥对和证书
	azkaban-web客户端使基于jetty web服务器的https访问，需要生成https访问的ssl密钥对（公钥/私钥）以及ssl证书，一般这种证书需要专业的证书品牌机构付费授权的，我们采用jdk提供的工具keytool生成测试用的秘钥对和证书（如果要商用就必须要授权的才行）
	①进入/opt/azkaban/azkaban-web-2.5.0根目录
	②使用命令：keytool -keystore keystore -alias jetty-ssl-key -genkey -kenalg RSA
        语法：keytool -keystore 【指定秘钥库的名称】 -alias 【别名，默认是mykey】-genkey -kenalg 【秘钥对算法，默认是DSA，这里使用RSA】
	之后输入秘钥库口令：123456，以及密钥口令：123456
	③检查/azkaban-web-2.5.0根目录生成keystore文件（其中包含秘钥对和证书）
⑦时间同步配置（具体参考Hadoop的ntp时间同步）
```

### 配置

##### 配置web服务器

```
①配置/opt/azkaban/azkaban-web-2.5.0/conf/azkaban.properties，
	#默认web 服务存放web文件的目录
    web.resource.dir=/opt/azkaban/azkaban-web-2.5.0/web
    #修改时区
    default.timezone.id=Asia/Shanghai
    #Azkaban UserManager class
    user.manager.class=azkaban.user.XmlUserManager
    #用户权限管理类的绝对路径
    user.manager.xml.file=/opt/azkaban/azkaban-web-2.5.0/conf/azkaban-users.xml

    #global文件的存放路径
    executor.global.properties=/opt/azkaban/azkaban-executor-2.5.0/conf/global.properties
    azkaban.project.dir=projects

    #配置mysql远程用户的信息
    database.type=mysql
    mysql.port=3306
    mysql.host=hadoop02
    mysql.database=azkaban
    mysql.user=alex
    mysql.password=alex123
    mysql.numconnections=100

    # Velocity dev mode
    velocity.dev.mode=false

    #配置jetty服务器
    jetty.maxThreads=25
    jetty.ssl.port=8443
    jetty.port=8081
    jetty.keystore=/opt/azkaban/azkaban-web-2.5.0/keystore
    jetty.password=123456
    jetty.keypassword=123456
    jetty.truststore=/opt/azkaban/azkaban-web-2.5.0/keystore
    jetty.trustpassword=123456
②web服务器用户配置，修改/opt/azkaban/azkaban-web-2.5.0/conf/azkaban-users.xml，
		<!-- 增加拥有admin权限的用户 -->
        <user username="admin" password="123456" roles="admin"/>
```

##### 配置Executor服务器

```
配置/opt/azkaban/azkaban-executor-2.5.0/conf/azkaban.properties,
		#时区
        default.timezone.id=Asia/Shanghai

        # Azkaban JobTypes Plugins
        azkaban.jobtype.plugin.dir=plugins/jobtypes

        #Loader for projects
        executor.global.properties=/opt/azkaban/azkaban-executor-2.5.0/conf/global.properties
        azkaban.project.dir=projects

        #mysql数据库配置
        database.type=mysql
        mysql.port=3306
        mysql.host=hadoop02
        mysql.database=azkaban
        mysql.user=alex
        mysql.password=alex123
        mysql.numconnections=100
```

### 启动

##### 启动Executor服务器

```
在目录/opt/azkaban/azkaban-executor-2.5.0/bin下执行：
	azkaban-executor-start.sh
```

##### 启动Web服务器

```
在目录/opt/azkaban/azkaban-web-2.5.0/bin下执行：
	azkaban-web-start.sh
```

##### 验证

```
①jps查看：AzkabanExecutorServer和AzkabanWebServer服务已经启动
②访问https://hadoop02:8443,进入azkaban web管理页
	账号和密码配置在azkaban-users.xml中
```

### 案例

##### 案例一：多任务工作流

```
需求分析：
	①依次开启mysql服务、hdfs集群服务、yarn集群服务、hive服务
	②hdfs依赖mysql，yarn依赖mysql
	③hive依赖hdfs、yarn
```

```
具体实现:
	①在window桌面
		①新建文件start-mysql.job,编辑：
            type=command
            command=service mysql start
    	②新建文件start-hdfs.job，编辑：
    		type=command
            command=ssh hadoop02 "source /etc/profile;$HADOOP_HOME/sbin/start-dfs.sh"
            dependencies=start-mysql
        ③新建文件start-yarn.job，编辑：
         	type=command
            command=ssh hadoop03 "source /etc/profile;$HADOOP_HOME/sbin/start-yarn.sh"
            dependencies=start-mysql
         ④新建文件start-hive.job，编辑：
          	type=command
            command=ssh hadoop02 "source /etc/profile;$HIVE_HOME/bin/hiveserver2"
            dependencies=start-hdfs,start-yarn
     ②将以上四个文件添加到打包成jobs.zip
     ③启动zakaban-executor-start.sh、zakaban-web-start.sh服务
     ④新建执行任务
     	①create project，命名jobs
     	②添加任务，导入jobs.zip
     ⑤执行任务，观察任务执行情况
```

##### 案例二：执行java程序任务

```
①准备一个项目的jar包，放到hadoop02的linux文件系统目录下
②在window桌面创建并编写jar.job文件
	type=javaprocess
	java.class=主类的全类名
	classpath=jar文件全路径
③将jar.job打包成jar.zip
④在azkaban-web管理平台创建任务，并导入jar.zip
⑤执行该任务
```

