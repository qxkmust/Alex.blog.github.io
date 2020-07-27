# Hadoop的部署

### 安装jdk

```
步骤一：

​		在官网下载jdk-8u241-linux-x64.tar.gz JDK8版本，通过ftp工具上传到linux /opt路径下

步骤二：

​		解压到/opt根目录，tar -zxvf jdk-8u241-linux-x64.tar.gz，之后进入/opt/jdk1.8.0_144/路径下，获取当前目录路径

步骤三：配置环境变量

​		编辑/etc/profile在最后一行添加全局变量配置：

​		export JAVA_HOME=/opt/jdk1.8.0_144

​		export PATH=$PATH:$JAVA_HOME/bin

步骤四：使配置立刻生效时间同步

​		source /etc/prifile
```

### 安装Hadoop

```
步骤一：

​		在官网下载hadoop-2.7.2.tar.gz版本，通过ftp工具上传到linux /opt路径下

步骤二：

​		解压到/opt根目录，tar -zxvf hadoop-2.7.2.tar.gz，之后进入/opt/hadoop-2.7.2路径下，获取当前目录路径

步骤三：配置环境变量

​		编辑/etc/profile在最后一行添加全局变量配置：

    ​		export HADOOP_HOME=/opt/hadoop-2.7.2

    ​		export PATH=$PATH:$HADOOP_HOME/bin

    ​		export PATH=$PATH:$HADOOP_HOME/sbin
		配置主机名与ip映射（因为下文中会用直接访问主机名）
			①修改主机名为hadoop01，/etc/hostname中增加hadoop01
			②修改主机名与ip映射关系，/etc/hosts中增加192.168.8.104 hadoop01

步骤四：使配置立刻生效

​		source /etc/prifile
```

### 部署Hadoop单节点（非分布式）应用

##### grep模式，用于大量的文本文件的快速匹配

```
	基本语法：hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar grep 输入路径 输出路径 正则过滤条件
	用法：从/input路径下的所有文件中检索以abc开头的字符串,将结果存放到output路径下
		hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar grep input/
		output 'abc[a-z]+'
	注意：
		①hadoop启动命令
        ②jar 以jar包运行方式运行
        ③share/hadoop/mapreduce-examles-2.7.2.jar  这是hadoop官网提供的案例目录
        ④grep 启动grep案例，官方提供的案例非常多，如wordcount模型等
        ⑤input 输入目录
        ⑥output 输出目录  （千万不要自己创建、hadoop源码中会判断这个文件是否存在、存在的话会报异常）
```

##### wordcount模式，用于大量文本文件的快速统计汇总

```
	基本语法：hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar wordcount 输入路径 输出路径
	用法：从/input路径下的所有文件中统计出每个单词的出现次数，将结果放到ouput路径下
		hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar wordcount input/ output
```

### 部署HDFS（单节点伪分布式）

```
HDFS简单理解就是存储文件的系统，就像百度云盘一样的东西。我们搭建好HDFS就相当于自己做了一个百度云盘，就可以将我们自己的一些文件上传到自己的HDFS中。
环境：
	使用VMWare中的主机hadoop01（192.168.8.104）作为节点
步骤一：基本配置
	①配置/etc/hadoop/hadoop-env.sh
		修改/opt/hadoop-2.7.2/etc/hadoop/hadoop-env.sh中：export JAVA_HOME=java的根目录
	②配置/etc/hadoop/core-site.xml
		在/opt/hadoop-2.7.2/etc/hadoop/core-site.xml的<configuration>节点中配置一下内容：
			<configuration>
                <!--指定HDFS中NameNode地址-->
                <property>
                        <name>fs.defaultFS</name>
                        <value>hdfs://hadoop01:9000</value>
                </property>
                <!--指定Hadoop运行时产生文件的存储目录-->
                <property>
                        <name>hadoop.tmp.dir</name>
                        <value>/opt/hadoop-2.7.2/data/tmp</value>
                </property>
            </configuration>
	③配置hdfs-site.xml
		/opt/hadoop-2.7.2/etc/hadoop/hdfs-site.xml中增加配置：
		<!--指定HDFS副本数量（默认是3），当前就只有一个hdfs主节点-->
			<property>
				<name>dfs.replication</name>
				<value>1</value>
			</property>
	

步骤二：启动集群
	①格式化NameNode（第一次启动时格式化，以后不需要）
		命令：bin/hdfs namenode -format
			会在hadoop根目录生成data文件夹
	②启动NameNode
		命令：sbin/hadoop-daemon.sh start namenode
	③启动DataNode
		命令：sbin/hadoop-daemon.sh start datanode
	④验证，登录节点ip:50070可以访问到hadoop监控
			也可以查看java进程中的namenode和datanode是否启动，jps

步骤三：运行程序（可以跑类似单节点HDFS中的程序，也可以执行简单的dfs命令）
	一：常用的dfs命令：
        ①查看指定目录下内容，hdfs dfs –ls [文件目录]
        ②查看文件内容，hdfs dfs –cat [file_path]
        ③上传文件到hdfs，hdfs dfs –put [本地地址] [hadoop目录]
        ④将hadoop上某个文件down至本地已有目录下，hadoop dfs -get [文件目录] [本地目录]
        ⑥删除hadoop上指定文件，hdfs  dfs –rm [文件地址]
        ⑦删除hadoop上指定文件夹（包含子目录等），hdfs dfs –rm [目录地址]
        ⑧在hadoop指定目录内创建新目录(-p创建多级目录)，hdfs dfs –mkdir -p /user/alex
        ⑨在hadoop指定目录下新建一个空文件，hdfs dfs  -touchz  /user/new.txt
        ⑩将hadoop上某个文件重命名，hdfs dfs –mv 旧名字 新名字t
        11、将正在运行的hadoop作业kill掉，hadoop job –kill  [job-id]
        12.更多查看帮助，hdfs dfs -help
	二：运行MR（MapReduce）程序
		①在hdfs的跟目录下创建/usr/alex/input文件夹
			hdfs dfs -mkdir -p /usr/alex/input
		②将本地文件core-site.xml上传到/input目录
			hdfs dfs -put /opt/hadoop-2.7.2/etc/hadoop/core-site.xml /usr/alex/input/
		③与单节点的操作类似：
			①从/input路径下的所有文件中检索以abc开头的字符串,将结果存放到output路径下
                hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar grep /usr/alex/input/ /usr/alex/output 'abc[a-z]+'
            ②从/input路径下的所有文件中统计出每个单词的出现次数，将结果放到ouput路径下
                hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar wordcount /usr/alex/input/ /usr/alex/output
注意：
	①HDFS方式不同于单节点，在HDFS的配置下，使用单节点的几种模式都会报错
	②如果格式化前/data目录已经存在会报错，要手动删除/data
	③如果namenode和datanode进程启动成功，会分别在/opt/hadoop-2.7.2/data/tmp/dfs/name/current和/opt/hadoop-2.7.2/data/tmp/dfs/data/current中生成VERSION文件，其中的clusterID是相同的
	④如果多次格式化NameNode，可能会导致clusterID不一致
	⑤若要在HDFS单节点运行MR程序，去除/etc/hadoop/core-site.xml fs.defaultFS配置
```

### 部署YARN

```
步骤一：基本配置
	①配置yarn
        ①配置yarn-env.sh的JAVA_HOME
            修改/opt/hadoop-2.7.2/etc/hadoop/yarn-env.sh中：
                export JAVA_HOME=java根目录
        ②配置yarn-site.xml
            <!--配置NodeManager，指定MapReduce获取数据的方式为mapreduce_shuffle-->
            <property>
                <name>yarn.nodemanager.aux-services</name>
                <value>mapreduce_shuffle</value>
            </property>
            <!--指定ResourceManager所在的服务器主机名-->
            <property>
                <name>yarn.resourcemanager.hostname</name>
                <value>hadoop01</value>
            </property>
     ②配置mapreduce
         ①配置mapred-env.sh
            修改/opt/hadoop-2.7.2/etc/hadoop/mapred-env.sh中：
                export JAVA_HOME=java根目录
         ②修改mapred-site.xml.template文件名为mapred-site.xml
         	mv /opt/hadoop-2.7.2/etc/hadoop/mapred-site.xml.template /opt/hadoop-2.7.2/etc/hadoop/mapred-site.xml
         ③配置mapred-site.xml
         	修改/opt/hadoop-2.7.2/etc/hadoop/mapred-site.xml中：
         	<!--指定MR程序在YARN上运行,若要保证在HDFS伪分布单节点运行MR，值改为local-->
         		<property>
         			<name>mapreduce.framework.name</name>
         			<value>yarn</value>
         		</property>
步骤二：启动集群
	①启动前必须保证NameNode和DataNode已经启动
	②启动ResourceManger
		sbin/yarn-daemon.sh start resourcemanager
	③启动NodeManager
		sbin/yarn-daemon.sh start nodemanager
	④验证，登录节点ip:8088可以访问到hadoop监控
			也可以查看java进程中的namenode和datanode是否启动，jps
步骤三：运行程序（可以跑类似单节点HDFS中的程序，也可以执行简单的dfs命令）
	参考HDFS步骤三
步骤四：配置历史服务器
	为了查看程序的历史运行情况，需要配置历史服务器：
	①配置mapred-site.xml
		修改/opt/hadoop-2.7.2/etc/hadoop/mapred-site.xml中：
			<!--配置历史服务器地址-->
			<property>
				<name>mapreduce.jobhistory.address</name>
				<value>hadoop01:10020</value>
			</property>
			<!--配置历史服务器web端地址-->
			<property>
				<name>mapreduce.jobhistory.webapp.address</name>
				<value>hadoop01:19888</value>
			</property>
	②启动历史服务器
		sbin/mr-jobhistory-daemon.sh start historyserver
步骤五：配置日志的聚集
	为了将程序运行日志上传到HDFS，需要开启日志聚集功能，开启后方便查看程序的运行情况，方便开发调试
	①配置yarn-site.xml
		修改/opt/hadoop-2.7.2/etc/hadoop/yarn-site.xml，
		<!--开启日志聚集功能-->
		<property>
			<name>yarn.log-aggregation-enable</name>
			<value>true</value>
		</property>
		<!--日志保留时间秒-->
		<property>
			<name>yarn.log-aggregation.retain-seconds</name>
			<value>86400</value>
		</property>
	②先关闭HistoryServer、NodeManager、ResourceManager进程
	③重新启动NodeManager、ResourceManager、HistoryServer
	④执行MR程序
	④验证，登录节点ip:19888/jobhistory

补充：
	Yarn的运行机制：
        1、用户向YARN提交程序，以Map Reduce程序为例，Resource Manager(资源管理器)接收到客户端程序的运行请求
        2、Resource Manager分配一个Container(资源)用来启动Application Master(程序管理员)，并告知Node Manager(节点管理员)，要求它在这个Container下启动Application Master（MRAppMaster进程）
        3、Application Master启动后，向Resource Manager发起注册请求
        4、Application Master向Resource Manager申请资源
        5、取得资源后，根据资源，向相关的Node Manager通信，要求其启动程序
        6、Node Manager启动MR（MR对job的执行是由YarnChild进程完成）
        7、Node Manager不断汇报MR状态和进展给Application Master
        8、当MR全部完成时，Application Master向Resource Manager汇报任务完成，并注销自己
```

### 搭建HDFS完全分布式

```
环境：
	①当前有四台服务器（192.168.8.104-192.168.8.107），跑在VMWare虚拟机上，分别对应hadoop01、hadoop02、hadoop03、hadoop04的主机
准备工作：
	①hadoop01已经按照HDFS单节点配置完成，其他主机只需要修改ip和host映射，不需要手动修改HDFS配置（下文将通过批量同步工具xsync）
	②自定义编写远程同步shell脚本xsync：
		①内容：
```

```
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
         ②用法：将当前服务器/usr/loca/bin目录下的所有文件同步到别的服务器
         	xsync /usr/local/bin
```

```
	③脚本xsync放到/usr/local/bin目录（用于存放用户的可执行程序）下，为脚本授权chmod 777 xsync
	
步骤一：按照三个节点集群的部署规划
	##################################################
			hadoop02	hadoop03 		hadoop04
	##################################################
	HDFS	NameNode	DataNode		SecondaryNameNode
			DataNode					DataNode
	##################################################
	YARN	NodeManager	NodeManager		NodeManager
						ResourceManager
	##################################################
	注意：
		①hadoop02中部署NameNode，而它的辅助名称节点部署在hadoop04上，避免同时部署到一台服务器造成服务器性能下降。
		②HDFS集群每个HDFS节点都有且只有一个DataNode，所有节点公用一个名称节点NameNode和辅助名称节点SecondaryNameNode
		③YARN作为资源调度的管理者，每个节点都有且只有一个NodeManager，所有节点公用一个ResourceManager
		④综合考虑服务器的性能，每个服务器都有三个进程。
		⑤hadoop01作为主节点，hadoop02、hadoop03、hadoop04作为副节点
		
步骤二：集群配置
	①当前集群是一个主节点，三个副节点，DataNode副本数要改为3
		修改所有节点的/opt/hadoop-2.7.2/etc/hadoop/hdfs-site.xml中
		<!--指定HDFS副本的数量-->
                        <property>
                                <name>dfs.replication</name>
                                <value>3</value>
                        </property>
    ②配置hadoop02节点
        ①为节点hadoop02分配一个辅助名称服务，这个服务放到节点hadoop04上，避免hadoop02性能受影响
            修改hadoop02节点/opt/hadoop-2.7.2/etc/hadoop/hdfs-site.xml中
            <!--指定hadoop辅助节点主机配置-->
                <property>
                <name>dfs.namenode.secondary.http-address</name>
                <value>hadoop04:50090</value>
                </property>
		②配置yarn-site.xml，指定YARN的ResourceManager放到节点hadoop03上
			修改hadoop02节点/opt/hadoop-2.7.2/etc/hadoop/yarn-site.xml中
			 <!--指定ResourceManager所在的服务器主机名-->
                <property>
                        <name>yarn.resourcemanager.hostname</name>
                        <value>hadoop03</value>
                </property>
		③通过远程批量同步脚本xsync将hadoop02的配置分发到hadoop03和hadoop04服务器的相同目录
			xsync /opt/hadoop-2.7.2/etc/hadoop/
		④验证：查看文件分发情况
			去别的节点查看是否配置已经同步更新
			
步骤三：集群单点启动
	①清除hadoop02/hadoop03/hadoop04下/opt/hadoop-2.7.2/下的data和logs目录
	②依次启动hadoop02中的NameNode、DataNode进程，hadoop03中的DataNode进程，hadoop04中的DataNode进程
	③验证：登录节点hadoop02:50090可以访问到hadoop监控
	
步骤四：SSH无密登录配置
	①配置ssh
		①ssh基本语法：ssh 另一台服务器ip或者主机名
		②生成公钥和私钥
			①ssh-keygen -t rsa
				-t 指定秘钥类型为rsa
			②敲三次回车，会在/root/.ssh目录下生成公钥id_rsa.pub和私钥id_rsa
		③进入/root/.ssh/目录，将公钥拷贝到要免密登录的目标服务器
			ssh-copy-id 主机名/ip
		④在节点hadoop02、hadoop03、hadoop04上分别操作以上三步，保证三个节点两两之间都可以免密登录
		注意：
			#################################################
            known_hosts		记录ssh访问过的计算机的公钥
            id_rsa			生成的私钥
            id_rsa.pub		生成的公钥
            authorized keys 存放授权过的无密登录的服务器公钥
			#################################################

步骤五：群起集群
	①配置slaves
        ①在节点hadoop02中配置/opt/hadoop-2.7.2/etc/hadoop/slaves，其中存放的是所有的DataNode节点名称
            修改slaves，添加：
                hadoop02
                hadoop03
                hadoop04
            注意节点名称前后不能有空格，最后一行之后不能有换行
        ②同步所有节点配置文件到hadoop03、hadoop04
            ①进入目录/opt/hadoop-2.7.2/etc/hadoop
            ②xsync slaves/
	③启动集群
		①启动所有节点的NameNode和DataNode
            使用/sbin/start-dfs.sh命令，可以依次启动hadoop02中的NameNode、DataNode进程，hadoop03中的DataNode进程，hadoop04中的DataNode进程
            注意：
                ①在任意一个节点使用start-dfs.sh都可以启动集群，也可以使用stop-dfs.sh依次关闭集群
                ②必须保证各个节点之间都可以无密登录
         ②启动所有节点的NodeManager和ResourceManager
         	使用/sbin/start-yarn.sh命令，可以依次启动hadoop02中的NodeManager进程，hadoop03中的NodeManager、ResourceManager进程，hadoop04中的NodeManager进程
         	注意：只能在节点hadoop03上使用start-yarn.sh命令有效，因为ResourceManager部署在hadoop03
         	
步骤六：集群时间同步
	将hadoop02作为时间服务器，以它为主，其它机器每十分钟从时间服务器获取最新时间
	①配置时间服务器
		①在主机hadoop02上安装ntp服务，yum install -y ntp
		②修改配置/etc/ntp.conf
			①允许指定网段访问
				restrict 192.168.8.0 mask 255.255.255.0 nomodify notrap 
			②如果集群在局域网中，指定不使用互联网时间
				注释掉：
				#server 0.centos.pool.ntp.org iburst
                #server 1.centos.pool.ntp.org iburst
                #server 2.centos.pool.ntp.org iburst
                #server 3.centos.pool.ntp.org iburst
			③如果时间服务器节点丢失网络连接，保证可以采用本地时间作为时间服务器为集群中的其它节点提供时间同步
				server 127.127.1.0 #如果上面的服务器都无法同步时间，就和本地系统时间同步
				fudge 127.127.1.0 stratum 10 #配置同步时间的精度
			④设置硬件时间与系统时间一起同步
				在/etc/sysconfig/ntpd文件中增加SYNC_HWCLOCK=yes
			⑤重启ntpd服务
				service ntpd start
			⑥设置ntpd服务开机启动
				chkconfig ntpd on
				注意：如果开启启动无效，可能是chronyd服务已经设置了ntp开启启动，需要停掉chronyd服务
					systemctl disable chronyd
	②配置从机
		①在从机上安装ntp服务，yum install -y ntp
		②编写定时任务脚本,在/home/crontab目录（任意目录都可以）
				①使用命令crontab -e，编辑：*/10 * * * * /usr/sbin/ntpdate hadoop02
					每隔十分钟同步一次时间
				②查看是否加入到定时任务
					crontab -l
		③重启ntpd服务
		④设置开机自启
	③测试：
		①修改从机机器时间
			date -s "1999-01-01 12:12:12"
		②观察十分钟后从机时间是否与hadoop02的时间一致

扩展：
	①服役新数据节点（datanode），也就是在现有的集群上快速增加数据节点
		以下模拟添加过程（生产中是通过增加物理机）
		①在hadoop04主机上克隆hadoop05主机
		②配置hadoop05的主机名和ip，重启生效
		③删除新节点hadoop06下/opt/hadoop-2.7.2/下的/data和/logs目录，启动datanode进程
			/sbin/hadoop-daemon.sh start datanode
	②通过服役新数据节点可以很容易增加数据节点，但是安全性差，为了防止有人恶意破坏集群以及盗取数据，有黑白名单两种方式(黑白名单不能同时出现相同的主机名/ip，会报错)：
		①白名单（配置哪些节点允许加入集群）
            ①在主节点hadoop02，/opt/hadoop-2.7.2/etc/hadoop/下增加白名单配置文件dfs.hosts,编辑：
                hadoop02
                hadoop03
                hadoop04
            只允许hadoop02、hadoop03、hadoop04加入集群
            ②在/opt/hadoop-2.7.2/etc/hadoop/hdfs-site.xml中
                <property>
                    <name>dfs.hosts</name>
                    <value>/opt/hadoop-2.7.2/etc/hadoop/dfs.hosts</value>
                </property>
            ③分发hdfs-site.xml给其他节点
                xsync hdfs-site.xml
            ④刷新NameNode
                hdfs dfsadmin -refreshNodes
                没有在白名单中的节点将会被强制退役（DataNode进程将强制被杀掉，不推荐）
        ②黑名单（配置哪些节点不允许加入集群）
        	①在主节点hadoop02，/opt/hadoop-2.7.2/etc/hadoop/下增加黑名单配置文件dfs.hosts.exclude,编辑：
                hadoop05
            不允许hadoop05加入集群
            ②在/opt/hadoop-2.7.2/etc/hadoop/hdfs-site.xml中
                <property>
                    <name>dfs.hosts.exclude</name>
                    <value>/opt/hadoop-2.7.2/etc/hadoop/dfs.hosts.exclude</value>
                </property>
            ③分发hdfs-site.xml给其他节点
                xsync hdfs-site.xml
            ④刷新NameNode（重新读取hdfs-site.xml配置）
                hdfs dfsadmin -refreshNodes
                注意：
                	①与白名单不同，在黑名单中的节点将正常退役（后台会将hadoop05的数据拷贝到其他节点，然后再退役，生产中推荐这种）		
                	②为防止hadoop05退役后造成集群负载不均衡，使用命令：start-balancer.sh
            ④观察hadoop02:50070 Datanodes页面中，hadoop05状态变更为Decommissioned（已退役），但是hadoop05的进程还存在，只是不与集群交互了，需要手动停止DataNode进程
            	hadoop-daemon.sh stop datanode
```

### HDFS的知识点

```
HDFS的组成架构：
	①NameNode，就是Master，是数据的管理者，功能包括：
		①管理HDFS的名称空间
		②配置副本策略
		③管理数据块映射信息
		④处理客户端读写请求
	工作机制：
```

![NameNode工作机制](D:\Git\MyRepository\Alex.blog.github.io\Images\NameNode工作机制.png)

```
	②DataNode，就是Slave，NameNode下达命令，由DataName执行。
		①存储实际的数据库
		②执行数据块的读/写操作
	工作机制：
```

![DataNode工作机制](D:\Git\MyRepository\Alex.blog.github.io\Images\DataNode工作机制.png)

```
	③SecondaryNameNode，不是NameNode的热备，不能代替NameNode，只是辅助
		①辅助NameNode，分担工作量，比如定期合并Fsimage和Edits，并推送给NameNode
		②紧急情况下，可以辅助恢复NameNode(具体是通过Fsimage和Edits合并成元数据，借助元数据恢复NameNode)
	④Client
		①文件切分，文件上传HDFS时，Client将文件切分成一个个Block，然后进行上传Block
		②与NameNode交互，获取文件的位置信息
		③与DataNode交互，读取或者写入数据
		④Client提供一些命令来管理HDFS，比如NameNode格式化
		⑤Client可以通过一些命令来访问HDFS，比如对HDFS增删改查操作
优点：
	①高容错性
		①数据自动保存多个副本，默认是3个（通过hdfs-site.xml中dfs.replication属性设置）
		②当某个节点有网络故障，当故障解除后，可以自动从别的副本恢复到3个
	②适合处理大数据
		①数据规模大，能够处理GB/TB/PB级别的数据
		②文件数量规模，能够处理百万规模以上的文件数量
	③可构建在廉价机器，通过多副本机制，提高可靠性
缺点：
	①数据访问高延时，可能有秒级的延时
	②无法高效的处理大量小文件
		①大量的小文件会占用NameNode的内存（每个文件不论大小，都会占用150byte）
		②小文件存储的寻址时间会超过读取时间，违反HDFS的设计目标
	③不支持并发写入以及文件随机修改
		①一个文件不允许多个线程同时写
		②仅支持文件数据追加（append操作），不支持文件的随机修改
补充：
	①为避免内存中的NameNode因断电等因素丢失，NameNode的数据被持久化到硬盘，保存在/opt/hadoop-2.7.2/data/tmp/dfs/name/current的Fsimage文件中，Fsimage文件是HDFS系统元数据的一个永久检查点，包含HDFS系统的所有目录的序列化信息
	②为解决Fsimage与NameNode数据不一致（因为速度差别很大）的问题，会把NameNode在内存中的写操作追加到/opt/hadoop-2.7.2/data/tmp/dfs/name/current/Edits文件（效率很高，能保证数据一致）
	③每次NameNode启动的时候会将最新的Fsimage文件和编辑日志edits_inprogress加载内存中，然后把编辑日志的写操作执行一遍，保证内存中的元数据信息是最新的，这一过程就是合并
	④当达到合并的触发条件（CheckPoint）满足：
            ①定时时间到（默认每隔3600秒）
            ②Edits数据满了（默认100万次写）
		此时，由SecondaryNameNode将Fsimage和Edits合并成新的Fsimage文件替换掉旧的Fsimage
操作：
	①将Fsimage和Edits转化为可读的XML文件：
        ①进入/opt/hadoop-2.7.2/data/tmp/dfs/name/current，其中包含多个Fsimage和Edits文件，使用命令：			hdfs oiv -p XML -i Fsimage文件名 -o 输出XML文件路径
              hdfs oev -p XML -i Edits文件名 -o 输出XML文件路径
            将Fsimage和Edits文件转化为XML文件，方便阅读
        ②查看XML内容，Fsimage.xml文件中记录着NameNode的目录（元数据），Edtis.xml记录着最近的hdfs写操作
    ②CheckPoint设置：
    	①默认情况下SecondaryNameNode每隔3600秒执行一次合并操作（将edits_inprogress和最新的fsimage合并），可以在hdfs-default.xml中更改：
    		<!--指定创建CheckPoint的时间间隔为60秒-->
    			<property>
    				<name>dfs.namenode.checkpoint.period</name>
    				<value>60</value>
    			</property>
    	②默认情况下每隔60秒检查编辑日志edits_inprogress的操作次数是否达到100万次，如果达到也会进行合并操作，可以在hdfs-default.xml中更改：
    			<property>
    				<name>dfs.namenode.checkpoint.txns</name>
    				<value>1000000</value>
    			</property>
    			<property>
    				<name>dfs.namenode.checkpoint.check.period</name>
    				<value>60</value>
    			</property>
    ③解决小文件存档的四种方法
    	①HDFS的HAR文件（存档文件），是一个更高效的文件存档工具，原理是将小文件包装成HAR文件，多个小文件对NameNode来说只需要存一个元数据就够了，有效的减少NameNode的内存占用
    	步骤：
    		①在节点hadoop03启动YARN进程
    		②上传三个小文件test01.txt、test02.txt、test03.txt到/usr/alex/input目录
    		③将/usr/alex/input下的所有小文件归档成一个名叫input.har的归档文件，并存储到/usr/alex/output下
    			hadoop archive -archiveName input.har -p /usr/alex/input /usr/alex/output
    			等待YARN在后台进行归档
    		④验证：	
    			查看归档文件
    				hadoop fs -ls -R har:///usr/alex/output/input.har
    	②Sequence File，是由一系列的二进制kv组成，将文件名作为key，文件内容作为value，就可以将大量小文件合并成一个大文件
    	③CombineFileInputFormat，用于将多个小文件合并成到一个切片中
    	④开启JVM重用
    		默认情况下，一个MapTask或者一个ReduceTask都会派生一个JVM实例单独处理。JVM重用是指同一个JVM实例可以被同一个job重复使用N次，可以极大的提高性能
    		开启JVM重用：
    			在/opt/hadoop-2.7.2/etc/hadoop/mapred-site.xml中增加：
    				<property>
                     <name>mapreduce.job.jvm.numtasks</name>
                     <!--具体要看业务-->
                     <value>10</value>
                    </property>
```

### HDFS故障处理

```
故障一：NameNode数据丢失
	原因：可能由于网络故障/断电/交换机等设备问题导致/opt/hadoop-2.7.2/data/tmp/dfs/name下数据丢失
	模拟故障过程：
		①杀掉NameNode进程，kill -9 进程号
		②删除/opt/hadoop-2.7.2/data/tmp/dfs/name下的所有数据
			rm -rf /opt/hadoop-2.7.2/data/tmp/dfs/name/*
			
	解决：
    方法一：
		思路：由于SecondaryNameNode中记录着跟NameNode几乎相同的数据（前者比后者少了一个edits_inprogress，会丢失最近写操作的数据）
        解决过程：
        	①拷贝SecondaryNameNode进程所在的节点hadoop04下的/opt/hadoop-2.7.2/data/tmp/dfs/namesecondary的数据到/opt/hadoop-2.7.2/data/tmp/dfs/name
                scp -r root@hadoop04:/opt/hadoop-2.7.2/data/tmp/dfs/namesecondary/* /opt/hadoop-2.7.2/data/tmp/dfs/name/*
            ②重启NameNode
                sbin/hadoop-daemon.sh start namenode
    方法二：
    	思路：指定NameNode启动时的参数-importCheckpint，NameNode守护进程会自动将同级目录中的SecondaryNameNode中的数据拷贝到NameNode的目录，完成数据恢复
    	解决过程：
    		①修改hdfs-site.xml中
    			<!--指定创建CheckPoint的时间间隔为60秒-->
    			<property>
    				<name>dfs.namenode.checkpoint.period</name>
    				<value>60</value>
    			</property>
    			<!--指定检查的namenode目录-->
    			<property>
    				<name>dfs.namenode.name.dir</name>
    				<value>/opt/hadoop-2.7.2/data/tmp/dfs/name</value>
    			</property>
        	①拷贝SecondaryNameNode的/namesecondary目录放到与NameNode的/name同一级，并删除/namesecondary下的in_use.lock
                scp -r root@hadoop04:/opt/hadoop-2.7.2/data/tmp/dfs/namesecondary /opt/hadoop-2.7.2/data/tmp/dfs/
            ③导入检查点数据
            	bin/hdfs namenode -importCheckpoint
            	之后等待NameNode守护进程完成数据拷贝
            ④重启NameNode
                sbin/hadoop-daemon.sh start namenode
```

### HDFS集群安全模式

```
①查看安全模式
	/bin/hdfs dfsadmin -safemode get
②进入（开启）安全模式
	/bin/hdfs dfsadmin -safemode enter
	安全模式中，HDFSClient及HDFS Shell都不能对集群进行写操作
③离开（关闭）安全模式
	/bin/hdfs dfsadmin -safemode leave
④等待安全模式，指hdfs shell命令等待安全模式关闭后自动执行
	/bin/hdfs dfsadmin -safemode wait
	跟上一个或多个hdfs shell命令
	
演示等待安全模式：
	①进入安全模式
	②在节点hadoop02 /home下编写shell脚本，命名safemode.sh:
		#!/bin/bash
		hdfs dfsadmin -safemode wait
		hdfs dfs -put /home/logs/log_merge.log /usr/alex
		hdfs dfs -mv /usr/alex/log_merge.log /usr/alex/log.log
	③执行safemode.sh
		.safemode.sh
	④在另一个节点hadoop03下关闭安全模式
	⑤验证脚本的命令是否执行
```

### HDFS可靠性配置

```
①NameNode的本地目录可以配置成多个，存放相同的内容，增加NameNode的可靠性，即使一个目录数据被清空，其它备份目录也可以保证NameNode正常工作
配置步骤：
	①在节点hadoop02下，配置/opt/hadoop-2.7.2/etc/hadoop/hdfs-site.xml
		<property>
			<name>dfs.namenode.name.dir</name>
            <value>
            	file:///${hadoop.tmp.dir}/dfs/name1,file:///${hadoop.tmp.dir}/dfs/name2
            </value>
		</property>
	②将配置分发给hadoop03和hadoop04
		xsync hdfs-site.xml
	③停止集群，删除/data 和/logs
	④格式化集群并启动
		hdfs namenode -format
		. start-dfs.sh
②DataNode可靠性配置，DataNode也可以配置成多个目录，每个目录存储的数据不一样（数据不是副本，所有目录加起来是完整的数据）
配置步骤：
	①在hdfs-site.xml中增加
		<property>
			<name>dfs.datanode.data.dir</name>
            <value>
            	file:///${hadoop.tmp.dir}/dfs/data1,file:///${hadoop.tmp.dir}/dfs/data2
            </value>
		</property>
	②将配置分发给hadoop03和hadoop04
		xsync hdfs-site.xml
	③停止集群，删除/data 和/logs
	④格式化集群并启动
	⑤验证：
		①目录/opt/hadoop-2.7.2/data/tmp/dfs/下出现/data1和/data2两个目录
		②向集群中上传文件，hadoop fs /opt/mysql-8.0.19-1.el7.x86_64.rpm-bundle.tar /
		③查看/data1和/data2的数据目录下分别存放的小文件
```

### 回收站功能

```
回收站默认关闭（fs.trash.interval值为0）
①开启回收站，配置core-site.xml
	<property>
        <name>fs.trash.interval</name>
        <value>60</value>
    </property>
②设置可访问垃圾回收站的用户名称（默认是dr.who），配置core-site.xml
	<property>
        <name>hadoop.http.staticuser.user</name>
        <value>root</value>
    </property>
④分发配置
	xsync core-site.xml
④启动集群
	start-dfs.sh
⑤验证：
	①删除文件
	②访问/user/root/.Trash/Current查看是否有被删除的文件
```

### Hadoop中MR阶段MapTask和ReduceTask工作过程

```
MapTask阶段：总共分为以下5个步骤：
	①.read阶段：当客户端提交job的时候，ResourceManager根据附加的三个信息（切片信息job.split和jar包以及xml文件）去开启相应数量MapTask读取文件
	②.map阶段：系统默认的map方式是TextInputFormat，通过读取行来切割数据，以便后序数据处理。
	③.collect阶段：当map阶段结束后，将切割完成的数据输送到环形缓冲区并分区、排序，系统默认按照key排序，按字典顺序排序，排序方式是快排（快速排序）。
	④.溢写阶段：就是当环形缓冲区数据达到其大小的**80%**时（默认大小为100M），将数据写入磁盘。
	⑤.combiner阶段：意为数据归并，即将数据进行归并排序。可有可无。在适当时候使用combiner，可以减少网络传输数据量，优化Hadoop框架性能。
	
	补充：
		①MapTask数量 = 切片数，MapTask的数量可以根据切片大小和文件读取规则估算出来
		②切片大小与Block大小一致：
            ①在HDFS非分布式单节点（local环境），切片大小32mb
            ②在HDFS伪分布式单节点，切片大小128mb
            ③在HDFS完全分布式集群，切片大小128mb
            注意：切片大小可以通过setMaxInputSplitSize()指定
        ③切片规则：
            KeyValueTextInputFormat / TextInputFormat：按文件大小切，每个文件单独切片
            如果文件大小200mb，块大小128mb，切成两片，第一片128mb，第二片72mb
            如果文件大小10kb，单独切一片
            CombineFileInputFormat：按文件大小切，多个文件作为一个切片
            如果切片大小是32mb，文件1是31mb，剩下的100个文件都是10kb，切一片就够了
            如果切片大小是31mb，文件1是32mb，剩下的100个文件都是100kb，切两片
            NLineInputFormat，按照指定的行数进行切片，每个文件单独切片
            如果文件1有100行，文件2有30行，按照3行一个切片，总共切成34 + 10片
        ④切片数（对应MapTask数量）：
            KeyValueTextInputFormat / TextInputFormat，切片数 = 文件数
            CombineFileInputFormat，切片数 = 文件总大小 / 切片大小
            NLineInputFormat，切片数 = 总行数 / 指定几行一个切片
            CombineTextInputFormat，切片数 = 文件总大小 / 块大小

“ReduceTask阶段”可以分解为以下4个阶段：
	①.coyp阶段：通过远程数据拷贝方式，从MapTask中拷贝已处理好的数据到内存。注意：（MapTask和ReduceTask很大程度不再同一台服务器，这就是为什么需要序列化和反序列化（bean对象））。
	②.merge阶段：即合并阶段。当拷贝过来的多个小文件合并成一个大的文件。当达到内存数据量的阈值（内存大小）时，自动会将数据写入磁盘。
	③.sort阶段：就是按照某种需求比如分组排序，与meger阶段同时进行。
	④.reduce阶段：将数据写入到指定的分区文件中。
	
	补充：
		①ReduceTask的数量(决定了最终.part-r文件的个数),默认是1
		②通过setNumReduceTasks()指定ReduceTask数量
			①如果ReduceTask=0，表示没有Reduce阶段
			②如果设置了分区数为n，则一定要设置ReduceTask数量大于>n，否则报错			
				job.setPartitionerClass(CustomPartitioner.class)//指定分区规则
            	job.setNumReduceTasks(大于等于n的整数);//设置ReduceTask数量
        ③修改/opt/hadoop-2.7.2/etc/hadoop/mapred-site.xml中修改：
        	set mapreduce.job.reduces=15，也可以修改MR任务的ReduceTask数量
```

### HadoopHA高可用部署

```
HadoopHA又分为HDFS HA和YARN HA:
	①HDFS HA的实现有两种方式：
		①QJM(Quorum Journal Manager)
			①原理：
```

![Hadoop高可用QJM原理](D:\Git\MyRepository\Alex.blog.github.io\Images\Hadoop高可用QJM原理.png)

```
		原理描述：
		1、namenode之间通过一组 journal node 共享编辑日志，standby namenode接管后，需要读取整个编辑日志来与active namenode同步状态，并继续读取active namenode写入的新操作。

        2、datanode需要同时向这组active-standby namenode发送数据块处理报告，因为数据块的映射信息保存在namenode的内存中。

        3、客户端使用ZKFC（zookeeper failover-controller）来处理namenode失效问题，该进程运行在每个namenode上，通过heartbeat监测active namenode是否失效

        4、secondary namenode的角色被standby namenode取代，由standby namenode为active namenode设置check point

        5、QJM的实现没有使用zookeeper。但是在HA选举active namenode时，使用了zookeeper。

        6、在某些特殊情况下（如网速慢），可能发生故障转移，这时有可能两个namenode都是active（脑裂现象）QJM通过fencing（规避）来避免这种现象。
      
```

​	①手动故障转移的配置

```
	配置过程：
                按照三个节点集群的部署规划：
                    ##################################################
                            hadoop02	hadoop03 		hadoop04
                    ##################################################
                    HDFS	NameNode	NameNode		
                            DataNode	DataNode		DataNode
                    ##################################################
                    JN		JN			JN				JN
                    ##################################################
                    ZK		ZKCli		ZKCli			ZKCli
                    ##################################################
                    注意：JN
                ①在节点hadoop02（192.168.8.105）的路径/opt创建新的目录/hadoop-2.7.2-ha,将同级的/hadoop-2.7.2/拷贝到/hadoop-2.7.2-ha
                    cp -r /hadoop-2.7.2 /hadoop-2.7.2-ha
                ②进入/opt/hadoop-2.7.2-ha/etc/hadoop,配置core-site.xml
                	<!--指定HDFS-HA集群NameNode集群的地址-->
                    <property>
                        <name>fs.defaultFS</name>
                        <value>hdfs://mycluster</value>
                    </property>
                    <!--指定Hadoop运行时产生文件的存储目录-->
                    <property>
                        <name>hadoop.tmp.dir</name>
                        <value>/opt/hadoop-2.7.2-ha/data/tmp</value>
                    </property>
                    <!-- 配置Zookeeper集群服务器地址 -->
                    <property>
                      <name>ha.zookeeper.quorum</name>
                      <value>hadoop02:2181,hadoop03:2181,hadoop04:2181</value>
                    </property>
                ③配置hdfs-site.xml
                    <!--指定HDFS副本的数量-->
                            <property>
                                <name>dfs.replication</name>
                                <value>1</value>
                            </property>
                    <!--配置HDFS-HA集群的服务名-->
                            <property>
                                <name>dfs.nameservices</name>
                                <value>mycluster</value>
                            </property>
                    <!--配置active-standby分别对应的NameNode节点名称
                    同一nameservices下,最多只能有两个NameNode
                    -->
                            <property>
                                    <name>dfs.ha.namenodes.mycluster</name>
                                    <value>nn1,nn2</value>
                            </property>
                    <!--配置active和standby的NN节点RPC通信端口-->
                            <property>
                                <name>dfs.namenode.rpc-address.mycluster.nn1</name>
                                <value>hadoop02:8020</value>
                            </property>
                            <property>
                                <name>dfs.namenode.rpc-address.mycluster.nn2</name>
                                <value>hadoop03:8020</value>
                            </property>
                    <!--配置NN节点的http通信端口-->
                            <property>
                                    <name>dfs.namenode.http-address.mycluster.nn1</name>
                                    <value>hadoop02:50070</value>
                            </property>
                            <property>
                                    <name>dfs.namenode.http-address.mycluster.nn2</name>
                                    <value>hadoop03:50070</value>
                            </property>
                    <!--配置active-standby共享的edits存放目录-->
                            <property>
                                 <name>dfs.namenode.shared.edits.dir</name>
                                 <value>qjournal://hadoop02:8485;hadoop03:8485;hadoop04:8485/mycluster</value>
                            </property>
                    <!--配置Active-Standby的切换实现方式-->
                            <property>
                                <name>dfs.client.failover.proxy.provider.mycluster</name>
                                <value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
                            </property>
                    <!-- 配置JN节点的数据存放目录 -->
                            <property>
                              <name>dfs.journalnode.edits.dir</name>
                              <value>/opt/hadoop-2.7.2-ha/data/tmp/jn</value>
                            </property>
                    <!--关闭HDFS权限检查，不局限于root用户-->
                            <property>
                                  <name>dfs.permissions.enable</name>
                                  <value>false</value>
                             </property>
                    <!--配置SSH-->
                            <property>
                                  <name>dfs.ha.fencing.methods</name>
                                  <value>sshfence</value>
                             </property>
                             <property>
                                  <name>dfs.ha.fencing.ssh.private-key-files</name>
                                  <value>/root/.ssh/id_rsa</value>
                             </property>
                    <!-- 开启自动故障转移 -->
                            <property>
                                  <name>dfs.ha.automatic-failover.enabled</name>
                                  <value>true</value>
                             </property>
                 ③启动hdfs-ha集群
                 	①清除hadoop02和hadoop03，/opt/hadoop-2.7.2-ha下的/logs和/data目录
                    ②分别在节点hadoop02、hadoop03、hadoop04上，进入/opt/hadoop-2.7.2-ha,首先启动JN（JournalNode）
                        sbin/hadoop-daemon.sh start journalnode
                       或者使用批量工具hadoop-daemons.sh, 一次启动三个JN
                    ③NameNode格式化
                    	①首先格式化hadoop02节点的NameNode，将data/tmp/dfs/name然后拷贝给hadoop03,保证hadoop03和hadoop02公用一个NameNode的数据
                        	bin/hdfs namenode -format
                        ②单点启动nn1（hadoop02的NameNode）
                        	sbin/hadoop-daemon.sh start namenode
                        ③同步nn1的元数据给hadoop03，进入haodop03的/opt/hadoop-2.7.2-ha
                        	bin/hdfs namenode -bootstrapStandby
                        ④单点启动nn2
                        	sbin/hadoop-daemon.sh start namenode
                    ③启动DataNode
                     	sbin/hadoop-daemons.sh start datanode
                    ④此时nn1和nn2都是Standby状态（无法提供HDFS服务），将其中一个切换Active
                    	bin/hdfs haadmin -transitionToActive --forcemanual nn1
                    ⑤验证：
                        ①查看NameNode状态，hadoop02是Active，hadoop03是Standby
                            bin/hdfs haadmin -getServiceState nn1
		                ②登录hadoop02:50070，提供FS服务，而hadoop02:50070不提供FS服务
                   注意：
                        ①切换状态时，NameNode间会互相通信，需要保证所有的NN都是启动状态
                        ②如果一个NN已经是Active状态，另一个只能是Standby，不能切换为Active（Active只能有一个）
                        ③如果一个Active状态的nn1被杀死了，nn2想切换为Active会失败（因为nn2会尝试与nn1进行通信，征求nn1的同意才能切换），只有重新启动nn1的NameNode后，nn2才能再次切换
                      	
                  ③生产中可能发生的问题：
                  	①启动两个NN节点，状态都是Standby，需要手动将其中一个切换为Active，但是切换可能发生Split-Brain，导致另一个节点挂掉（生产中不会使用手动切换）
                  	②如果发生某个NN运行一段时间自动挂掉，检查logs/下zkfc日志，排查问题
```

​	②自动故障转移的配置

原理：

![HDFS-HA故障转移机制](D:\Git\MyRepository\Alex.blog.github.io\Images\HDFS-HA故障转移机制.png)

```
原理描述：
	①现有的两个NN节点nn1（Active）和nn2（Standby），nn1启动的时候会向zookeeper服务端注册一个临时节点zNode1，同时会有一个zookeeper的客户端zkfc1（zookeeper failover controller），它负责监听zNode1节点是否存活
	②如果nn1发生故障或者进程被杀掉，负责监控zNode1的zkfc1会立刻通知nn2的zkfc2，zkfc2为了防止同时出现两个Active的NN，会强行杀死nn1（通过hdfs-site.xml中配置的sshfence脚本），然后将nn2切换为Active状态
	③nn2同样在zookeeper服务端注册一个临时节点zNode2，由zkfc2进行监控
	④zkfc的作用：是监控RM的健康状态，并执行选举作用
```

```
    配置过程：
              	①自动故障转移依赖Zookeeper，基于以上hdfs-site.xml和core-site.xml配置，增加以下：
              	在hdfs-site.xml增加：
              		<!-- 开启自动故障转移 -->
                        <property>
                              <name>dfs.ha.automatic-failover.enabled</name>
                              <value>true</value>
                         </property>
              	在core-site.xml增加：
              		<!-- 配置Zookeeper集群服务器地址 -->
                        <property>
                          <name>ha.zookeeper.quorum</name>
                          <value>hadoop02:2181,hadoop03:2181,hadoop04:2181</value>
                        </property>
                 ②启动hdfs-ha集群
                 	①分别在hadoop02、hadoop03、hadoop04节点启动zookeeper服务
                 		zkServer.sh start
                 		或者使用批量脚本zkServer-batch-start一次开启所有
                 	②初始化hdfs-ha在Zookeeper中状态
                        bin/hdfs zkfc -formatZK
                      相当于在zookeeper集群新增了临时节点/hadoop-ha/mycluster
                    ③启动HDFS集群
                        ①启动集群，在节点hadoop02（任意节点都行），使用命令
                        	sbin/start-dfs.sh
                        ②start-dfs.sh启动了集群中的所有NameNode,DataNode,ZKFC,JN
                    ④验证
                 		①登录hadoop02:50070/hadoop03:50070，其中一个状态是Standby，另一个是Active,哪个NN所在节点上的ZKFC先启动，哪个NN就是Active
                 		②使用自定义Shell脚本jps-branch查看所有节点java进程状态:
                                ##########hadoop02 java进程#################
                                102453 Jps
                                86804 DFSZKFailoverController (*)
                                80552 QuorumPeerMain (*)
                                86283 NameNode (*)
                                86412 DataNode
                                86620 JournalNode (*)
                                ##########hadoop03 java进程#################
                                32242 Jps
                                123748 DataNode 
                                27205 DFSZKFailoverController (*)
                                27654 NameNode (*)
                                122150 QuorumPeerMain
                                123852 JournalNode (*)
                                ##########hadoop04 java进程#################
                                126964 DataNode
                                125851 QuorumPeerMain (*)
                                127051 JournalNode (*)
                                2078 Jps
`							注意必须保证(*)进程正常运行
					③生产中可能发生的问题：
						①两个NN都是Standby，可能是ZKFC进程没有起来，重启ZKFC
						 	sbin/hadoop-daemon.sh stop zkfc
						 	sbin/hadoop-daemon.sh start zkfc
                 		②如果运行一段时间NN挂掉了，按照第一步，重启ZKFC,之后重启NameNode
                 		③如果强制杀掉nn1的Active，可能nn2无法自动切换为Active，查看namenode的日志找到原因：hadoop HA sshfen切换隔离时无法跳转ssh: bash: fuser: 未找到命令，Centos Mininal版本没有是没有fuser命令，需要安装：yum install psmisc
```

### YARN高可用配置

```
YARN-HA高可用与HDFS-HA类似，同样依赖zookeeper
	配置过程：
		在hadoop02、hadoop03、hadoop04三个节点部署YARN-HA集群：
		①部署规划：
                    ##################################################
                            hadoop02	hadoop03 		hadoop04
                    ##################################################
                    HDFS	NameNode	NameNode		
                            DataNode	DataNode		DataNode
                    ##################################################
                    JN		JN			JN				JN
                    ##################################################
                    ZK		ZKCli		ZKCli			ZKCli
                    ##################################################
                    YARN	NodeManager	 NodeManager	NodeManager
                         ResourceManager ResourceManager
                    ##################################################
         ②具体配置：
         	①在hadoop02节点，修改/opt/hadoop-2.7.2-ha/etc/hadoop/yarn-site.xml,
         		<!-- 开启YARN高可用 -->
                    <property>
                    <name>yarn.resourcemanager.ha.enabled</name>
                    <value>true</value>
                    </property>
                <!--指定ResourceManager集群的名称-->
                    <property>
                    <name>yarn.resourcemanager.cluster-id</name>
                    <value>cluster-yarn</value>
                    </property>
                <!-- 配置RM集群中节点名称 -->
                    <property>
                    <name>yarn.resourcemanager.ha.rm-ids</name>
                    <value>rm1,rm2</value>
                    </property>
                <!-- rm1对应的主机名 -->
                    <property>
                    <name>yarn.resourcemanager.hostname.rm1</name>
                    <value>hadoop02</value>
                    </property>
                <!-- rm2对应的主机名 -->
                    <property>
                    <name>yarn.resourcemanager.hostname.rm2</name>
                    <value>hadoop03</value>
                    </property>
                <!-- 配置Zookeeper集群服务器地址 -->
                    <property>
                    <name>yarn.resourcemanager.zk-address</name>
                    <value>hadoop02:2181,hadoop03:2181,hadoop04:2181</value>
                    </property>
                <!-- 开启自动故障转移 -->
                    <property>
                    <name>yarn.resourcemanager.recovery.enabled</name>
                    <value>true</value>
                    </property>
                <!-- 指定RM集群节点的状态信息存储在zookeeper集群 -->
                    <property>
                    <name>yarn.resourcemanager.store.class</name>
                    <value>org.apache.hadoop.yarn.server.resourcemanager.recovery.FileSystemRMStateStore</value>
                    </property>
        ③启动yarn-ha集群
        	①在hadoop02、hadoop03、hadoop04节点启动zookeeper服务
        		使用自定义脚本zkServer-batch-start
        	③启动HDFS集群
        		在hadoop02节点，进入/opt/hadoop-2.7.2-ha，使用命令：
        			sbin/start-dfs.sh
        	③启动yarn集群
        		①在hadoop02，进入/opt/hadoop-2.7.2-ha，使用命令：
        			sbin/start-yarn.sh
        		②在hadoop03使用单节点启动RM
        			sbin/yarn-daemon.sh start resourcemanager
        	④验证：
        		①查看rm1和rm2的状态，一个是Active，另一个是Standby
        			bin/yarn rmadmin -getServiceState rm1
        			bin/yarn rmadmin -getServiceState rm2
        		②如果hadoop02上的rm1是Active状态，手动杀掉rm1，登录hadoop03:8088,如果正常登录，说明rm2已经切换为Active状态
```

### DN空间满了怎么办

```
如果DN空间满了，会导致hdfs永久进入安全模式，无法进行写操作，解决办法：
方法一：手动清除垃圾站（hdfs://user/root/.Trash）的数据
	①强制hdfs退出安全模式：
		hdfs dfsadmin -safemode leave
	②手动清除Trash
		hadoop fs -expunge
方法二：自动清除垃圾站（修改core-site.xml）
	①fs.trash.interval，默认1440分钟（1天）清理一次Trash
	②fs.trash.max.percentused，当已使用空间率大于该值, 执行回收以释放空间. 默认为0.8f
方法三：服务器动态扩容增加硬盘，生产中都会配置DataNode多目录
```

### 元数据丢失怎么办

```
如果误删了hdfs元素的目录，导致hdfs的块丢失，进一步导致NN进入安全模式，不能进行写操作，解决办法：
步骤一：退出安全模式
	hdfs dfsadmin -safemode leave
步骤二：检查hdfs丢失的块
	hdfs fsck /
	注意：可查看丢失的块的个数，以及丢失的数据大小
步骤三：删除丢失的块
	如果块丢失了，一般重启hdfs服务，2NN会自动帮助NN恢复，如果恢复不了，就只能删除丢失的块
	hdfs fsck -delete
```

### 支持LZO和Snappy压缩

```
hadoop默认支持GzipCodec、 DefaultCodec、BZip2Codec，需要自己编译Snappy和LZO的组件，并在/opt/hadoop-2.7.2/etc/hadoop/core-site.xml增加：

<!-- 增加lzo和Snappy压缩方式，默认只支持GzipCodec、 DefaultCodec、BZip2Codec-->
<property>
  <name>io.compression.codecs</name>
  <value>
    org.apache.hadoop.io.compress.GzipCodec,
    org.apache.hadoop.io.compress.DefaultCodec,
    org.apache.hadoop.io.compress.BZip2Codec,
    com.hadoop.compression.lzo.LzoCodec,
    com.hadoop.compression.lzo.LzopCodec,
    org.apache.hadoop.io.compress.SnappyCodec
  </value>
</property>
<!-- 指定lzop的类-->
<property>
  <name>io.compression.codec.lzo.class</name>
  <value>com.hadoop.compression.lzo.LzoCodec</value>
</property>
<!-- 指定Snappy的类-->
<property>
  <name>io.compression.codec.snappy.class</name>
  <value>org.apache.hadoop.io.compress.SnappyCodec</value>
</property>
```

### 脚本工具

```
xshell.sh脚本，作用在以下三台主机同步进行查询操作，比如xshell.sh date
#!/bin/bash
#"同步查询集群进程"
for i in hadoop02 hadoop03 hadoop04
do
        echo "############## $i ###############"
        ssh $i "$*"
done

```

