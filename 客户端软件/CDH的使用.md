# CDH的使用

### 准备工作

```
步骤一：
	在三台虚拟机192.168.8.108-cdh-hadoop-02、192.168.8.109-cdh-hadoop-03、192.168.8.110-cdh-hadoop-04分别安装Centos6.5系统，服务部署的规划：
	########################################################
	hadoop02				hadoop03				hadoop04
	########################################################
	CM Server/CM Agent		CM Agent				CM Agent
	########################################################
	注意：这里是使用的Centos6.5系统
步骤二：
	①分别配置三台虚拟机的ip和hostname，hadoop02(192.168.8.108),hadoop03(192.168.8.109),hadoop04(192.168.8.110)
	②配置ssh免密
		①生成公钥和私钥
			①ssh-keygen -t rsa
				-t 指定秘钥类型为rsa
			②敲三次回车，会在/root/.ssh目录下生成公钥id_rsa.pub和私钥id_rsa
		②进入/root/.ssh/目录，将公钥拷贝到要免密登录的目标服务器
			ssh-copy-id 主机名/ip
		③在节点hadoop02、hadoop03、hadoop04上分别操作以上三步，保证三个节点两两之间都可以免密登录
步骤三：编写集群同步脚本xsync
	#!/bin/sh
    # 获取输入参数个数，如果没有参数，直接退出
    pcount=$#
    if((pcount==0)); then
    echo no args...;
    exit;
    fi

    # 获取文件名称
    p1=$1
    fname=`basename $p1`
    echo fname=$fname
    # 获取上级目录到绝对路径
    pdir=`cd -P $(dirname $p1); pwd`
    echo pdir=$pdir
    # 获取当前用户名称
    user=`whoami`
    # 循环，将当前服务器指定目录下的文件同步到主机03/04
    for((host=3; host<=4; host++)); do
    echo $pdir/$fname $user@slave$host:$pdir
    echo ==================slave$host==================
    rsync -rvl $pdir/$fname $user@hadoop0$host:$pdir
    done
    #Note:这里的@hadoop0$host是拼接主机名，需要做相应修改。另外，for循环中的host的边界值由自己的主机编号决定。
步骤四：在三台主机安装JDK
	使用jdk-8u251-linux-x64.tar.gz，上传并解压到/opt目录，添加到环境变量
步骤五：在三台主机安装MYSQL
	①使用mysql5.5.52版本，将MySQL-server-5.5.52-1.el6.x86_64.rpm和MySQL-client-5.5.52-1.el6.x86_64.rpm上传到/opt目录，rpm安装
	②开启mysql服务
	③登录mysql客户端，删除mysql.user表中user为空的记录，只保留user=root,host=localhost
	④修改root用户密码并增加权限
		①mysql5版本，使用命令update user set password = password('root') where user = 'root'
		②授权root用户可使用新的密码登录：
			grant all privileges on *.* to '用户名'@'主机' identified by 'root' with grant option;
	⑤创建远程访问用户并增加权限
		①新增用户：create user alex identified by 'alex123'
        ②授权：
            ①grant all privileges on *.* to 'alex'@'%' identified by 'alex123' with grant option;
            ②flush privileges
步骤六：将jdbc依赖拷贝到/opt/cm-5.12.1/share/cmf/lib目录
步骤七：在hadoop02主机创建CM用的数据库
	登录mysql客户端，依次创建监控数据库、Hive数据库、Oozie数据库、Hue数据库
	create database amon DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
	create database hive DEFAULT CHARSET latin1 COLLATE latin1_general_ci;
	create database oozie DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
	create database hue DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
步骤八：在三台主机安装第三方依赖
	yum -y install chkconfig python bind-utils httpd psmisc libxslt zlib sqlite cyrus-sasl-plain cyrus-sasl-gssapi fuse fuse-libs redhat-lsb
步骤九：在三台主机关闭SELINUX
	修改配置文件/etc/selinux/config：
	SELINUX=enforcing 改为 SELINUX=disabled
步骤十：配置ntp时间同步（参考Hadoop）
步骤十一：关闭三台服务器的防火墙，配置ssh免密登录
```

### 下载安装

```
步骤一：下载
	①下载CDH压缩包：
		登录CDH版本库http://archive.cloudera.com/cm5/cm/5/，下载对应linux系统的CDH版本，这里使用cloudera-manager-el6-cm5.12.1_x86_64.tar.gz
	②非必须，下载离线库：
		登录CDH离线库http://archive.cloudera.com/cdh5/parcels/5.12.1/，下载对应的5.12.1版本的离线库
		CDH-5.12.1-1.cdh5.12.1.p0.3-el6.parcel
		CDH-5.12.1-1.cdh5.12.1.p0.3-el6.parcel.sha
		manifest.json
步骤二：安装
	将cloudera-manager-el6-cm5.12.1_x86_64.tar.gz上传到hadoop02、hadoop03、hadoop04下的/opt目录，并解压出两个文件夹cloudera和cm-5.12.1，将cloudera移动到cm-5.12.1目录中。以下是CDH5的目录介绍：
        /var/log/* : 相关日志文件（相关服务的及CM的）。
        /usr/share/cmf/ : 程序安装目录。
        /usr/lib64/cmf/ : Agent程序代码。
        /var/lib/cloudera-scm-server-db/data : 内嵌数据库目录。
        /usr/bin/postgres : 内嵌数据库程序。
        /etc/cloudera-scm-agent/ : agent的配置目录。
        /etc/cloudera-scm-server/ : server的配置目录。
        /opt/cloudera/parcels/ : Hadoop相关服务安装目录。
        /opt/cloudera/parcel-repo/ : 下载的服务软件包数据，数据格式为parcels。
        /opt/cloudera/parcel-cache/ : 下载的服务软件包缓存数据
	
步骤三：配置
	①在三台主机创建名称为cloudera-scm的用户：
        useradd
        --system	//创建系统用户
        --home=/opt/cm-5.12.1/run/cloudera-scm-server	//指定home目录
        --no-create-home	//不使用默认的/home目录
        --shell=/bin/false  //用户不具有任何权限
        --comment "cloudera-scm" 	//用户描述
        cloudera-scm 	//自定义的用户名
     ②配置CM Agent的服务端地址
     	修改/opt/cm-5.12.1/etc/cloudera-scm-agent/config.ini:
     		server_host=hadoop02
     ③创建CM数据库
         /opt/cm-5.12.1/share/cmf/schema/scm_prepare_database.sh
         mysql	//使用mysql数据库
         cm		//数据库名称
         -hhadoop02	//远程连接的主机名/ip
         -ualex		//用户名
         -palex123	//密码
         --scm-host hadoop02	//CM Server的主机名/ip
         scm	//？固定写法
         scm	//？固定写法
         scm	//？固定写法
     ④创建.parcel文件仓库的所在目录parcel-repo
     	mkdir -p /opt/cloudera/parcel-repo
     ⑤非必须，上传离线库文件到
     	将以下文件上传到/opt/cloudera/parcel-repo
            CDH-5.12.1-1.cdh5.12.1.p0.3-el6.parcel
            CDH-5.12.1-1.cdh5.12.1.p0.3-el6.parcel.sha
            manifest.json
        注意：CM监控页也可以在线安装CDH-parcel，需要在代理中配置要下载的CDH版本地址：http://archive.cloudera.com/cdh5/parcels/5.12.1/
     ⑥创建parcels目录，并更改用户及用户组
     	mkdir /opt/cloudera/parcels
     	chown cloudera-scm:cloudera-scm /opt/cloudera/parcels
	 ⑦分发到hadoop03、hadoop04
	 	xsync /opt/cm-5.12.1
	 	xsync /opt/cloudera
步骤四：非必须，编写脚本
	①编写启动CM Server服务脚本，创建/usr/local/bin/cm-server.sh
#!/bin/bash
#开启/关闭hadoop02上的cloudera-scm-server服务

case $1 in
"start"){
echo "######### 启动$i cloudera-scm-server服务 ###########"
/opt/cm-5.12.1/etc/init.d/cloudera-scm-server start
};;
"stop"){
echo "######### 停止$i cloudera-scm-server服务 ###########"
/opt/cm-5.12.1/etc/init.d/cloudera-scm-server stop
};;
esac
	②编写启动/关闭CM Agent服务脚本，创建/usr/local/bin/cm-agent.sh
#!/bin/bash
#群起/群关cloudera-scm-agent服务
#由于无法关闭CM Agent进程，"stop"相当于重启
case $1 in
"start"){
for i in hadoop02 hadoop03 hadoop04
do
echo "######### 启动$i cloudera-scm-agent服务 ###########"
ssh $i "ps -ef|grep cloudera-scm-agent|grep -v 'grep'|grep -v logfile|awk '{print \$2}'|xargs kill"
ssh $i "/opt/cm-5.12.1/etc/init.d/cloudera-scm-agent start"
done
};;
"stop"){
for i in hadoop02 hadoop03 hadoop04
do
echo "######### 停止$i cloudera-scm-agent服务 ###########"
ssh $i "/opt/cm-5.12.1/etc/init.d/cloudera-scm-agent stop"
done
};;
esac
    ③编写查询CM服务PID脚本，创建/usr/local/bin/cm-pid.sh
        #!/bin/bash
        #查看cloudera-scm-server cloudera-scm-agent的pid

        case $1 in
        "server"){
                echo "######## hadoop02 cloudera-scm-server进程 #########"
                ssh hadoop02 "ps -ef|grep 'cloudera-scm-server'|grep -v 'grep'|grep -v 'logfile'|awk '{print \$2}'"
        };;
        "agent"){
                for i in hadoop02 hadoop03 hadoop04
                do
                        echo "######## $i cloudera-scm-agent进程 #########"
                        ssh $i "ps -ef|grep 'cloudera-scm-agent'|grep -v 'grep'|grep -v 'logfile'|awk '{print \$2}'"
                done
        };;
        esac
    ④给以上脚本授权755
步骤五：启动CM服务
	①启动CM Server服务
		cm-server.sh start
	②启动CM Agent服务
		cm-agent.sh start
步骤六：验证
	启动CM服务，等待大概3分钟，登录CM监控页面：http://192.168.8.108:7180/cmf/home（用户名和密码都是admin）
步骤七：集群部署
	在CM监控页面进行集群部署：
	-> 选择免费
	-> 继续
	-> 选定集群的主机
	-> 选择CDH的版本（默认）
	-> 等待下载安装（自动将.parcel文件和.sha文件下载到/opt/cloudera/parcel-repo目录）
	
	可能存在的问题：
	①在安装完CDH.parcel文件后，会自动检查主机的健康状态，从日志中发现“无法检测JAVA_HOME”的异常，原因：CDH会扫描/usr/java目录下的jdk1.7或者jdk1.8，而不会扫描系统环境变量
	解决：手动创建/usr/java，在该目录下创建指向/opt/jdk1.8.0_251目录的软连接
		ln -s /opt/jdk1.8.0_251 /usr/java/jdk1.8
	②若某个主机出现“主机运行状态不良情况”导致组件未下载/安装完成
	解决：①删除问题主机/opt/cm-5.16.1/lib/cloudera-scm-agent/cm_guid文件
		②单独重启问题主机上的CM Agent服务，/opt/cm-5.12.1/etc/init.d/cloudera-scm-agent restart
		③重新登录CM监控页，完成安装
更多问题，参考https://jingyan.baidu.com/article/da1091fb761f42027949d661.html
```

### 安装Hadoop组件

#### 安装规划

| 安装次序<br>由上到下 | hadoop02                           | hadoop03       | hadoop04       |
| -------------------- | ---------------------------------- | -------------- | -------------- |
| HDFS                 | NN/DN                              | DN             | DN/2NN         |
| YARN                 | NM                                 | RM/NM          | NM             |
| ZK                   | QuorumPeerMain                     | QuorumPeerMain | QuorumPeerMain |
| KAFKA                | kafka                              | kafka          | kafka          |
| FLUME                | Agent                              | Agent          | Agent          |
| HIVE                 | HiveServer2、Hive Metastore Server | /              | /              |
| Oozie                | Oozie Server                       | /              | /              |
| Hue                  | Hue Server、Load Balancer          | /              | /              |

安装完CM后，从自定义服务中选择hadoop组件，按照规划图在相应的主机安装服务

##### 安装HDFS

```
从自定义服务中选择hadoop组件
注意：
	①在安装完HDFS服务后，为了读写方便，在配置中关闭HDFS的权限检查：dfs.permissions，实际生产中可根据需求开启权限控制（比如为特定用户分配读写权限）
	②配置HDFS支持LZO压缩，配置 -> 添加压缩编解码器：com.hadoop.compression.lzo.LzopCodec
	③将mysql-connector-java.jar驱动导入到/opt/cloudera/parcels/CDH-5.12.1-1.cdh5.12.1.p0.3/lib/hadoop/lib/mysql-connector-java-5.1.48.jar，将jar分发给hadoop03和hadoop04
	注意：主机必须支持lzo（安装lzo-native库）：
		yum install -y lzo
```

##### 安装YARN

```
从自定义服务中选择hadoop组件
注意：
	①配置YARN支持LZO压缩，配置classpath -> 指定jar包：/opt/cloudera/parcels/HADOOP_LZO/lib/hadoop/lib/hadoop-lzo-cdh4-0.4.15-gplextras.jar
	②根据需求，适当增加yarn.nodemanager.resource.memory-mb
	③主机必须支持lzo（安装lzo-native库）：
		yum install -y lzo
```

##### 安装ZK

```
添加服务 -> 从自定义服务中选择hadoop组件
```

##### 安装KAFKA

```
主机 -> Parcel -> 在线下载、安装Parcel组件，默认安装就行
```

#####  安装FLUME

```
添加服务 -> 从自定义服务中选择hadoop组件
注意：flume agent不需要指定外源配置文件，在->配置->配置文件中进行输入
```

##### 安装HIVE

```
添加服务 -> 从自定义服务中选择hadoop组件
注意：
	①配置hive支持LZO压缩，配置 -> Hive辅助JAR目录：/opt/cloudera/parcels/HADOOP_LZO/lib/hadoop/lib/
	②配置hive metastore自动建表
		datanucleus.autoCreateSchema = true
	③将mysql-connector-java.jar驱动导入到/opt/cloudera/parcels/CDH-5.12.1-1.cdh5.12.1.p0.3/lib/hive/lib/mysql-connector-java-5.1.48.jar
```

#####  安装OOZIE

```
添加服务 -> 从自定义服务中选择hadoop组件
注意：
	①oozie界面下->安装oozie共享库
	③将mysql-connector-java.jar驱动导入到/opt/cloudera/parcels/CDH-5.12.1-1.cdh5.12.1.p0.3/lib/oozie/lib/mysql-connector-java-5.1.48.jar
```

##### 安装Sqoop

```
添加服务 -> 从自定义服务中选择hadoop组件
注意：
	①将mysql-connector-java.jar驱动导入到/opt/cloudera/parcels/CDH-5.12.1-1.cdh5.12.1.p0.3/lib/sqoop/lib/mysql-connector-java-5.1.48.jar，将jar分发给hadoop03和hadoop04
```

##### 安装HUE

```
添加服务 -> 从自定义服务中选择hadoop组件
注意：
	配置HUE麻烦，要做些准备工作：
		①为LoadBalancer服务所在的hadoop02主机安装mod_ssl服务
			yum install -y mod_ssl
		②安装libmysqlclient_r.so.16依赖库
			rpm -ivh MySQL-shared-compat-5.5.52-1.el6.x86_64.rpm
		③如果安装HUE还是报错，需要安装libxslt-devel
			yum install -y libxslt-devel
		④如果使用HUE WEB UI报错：【Could not start SASL: Error in sasl_client_start (-4) SASL(-4): no mechanism available】，需要增加以下依赖，之后重启HUE服务
			yum install cyrus-sasl-plain cyrus-sasl-devel cyrus-sasl-gssapi  cyrus-sasl-md5
```

### CDH权限问题

```
cdh各组件在安装时会创建多种用户和系统组，比如hdfs、hive、sqoop等，导致不同用户和系统组之间进行数据交换会发生权限不足的问题，解决办法：
方法一：修改各个组件的“用户"和"系统组"为root
	在各个组件 -> 配置 -> 搜索“用户”和“系统组”，全部更改为root
注意：
	生产中会使用权限控制组件Ranger对hdfs进行权限控制
```

