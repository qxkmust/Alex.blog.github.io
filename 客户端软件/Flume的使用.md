# Flume的使用

### 下载安装

```
步骤一：
	从官网http://flume.apache.org/download.html下载相应版本的gz包，本机使用的apache-flume-1.7.0-bin.tar.gz
步骤二：
	使用sftp工具上传到VMWare主机hadoop02（192.168.8.105）的/opt目录并解压，重命名解压后的目录名称flume-1.7.0
步骤三：配置信息
	①在主机hadoop02（192.168.8.105），进入/opt/flume-1.7.0/bin,将当前目录配置到系统环境变量中：
        ①编辑/etc/profile文件，增加
                export FLUME_HOME=/opt/flume-1.7.0
                export PATH=$PATH:$FLUME_HOME/bin
		②使配置立刻生效,source /etc/profile
    ②保证JAVA_HOME已经加入环境变量
步骤四：启动flume服务
	flume-ng <command> [options]...
    commands:
      help                      display this help text
      agent                     run a Flume agent
      avro-client               run an avro Flume client
      version                   show Flume version info
	具体启动见下文
```

### Flume使用

##### 案例一：监控端口数据

```
需求分析：
	①在主机hadoop03上通过netcat客户端向服务端2345端口发送数据
	②在主机hadoop02上启动flume监控本机2345端口的数据
	②flume的netcat tcp source组件读取数据，数据通过memory channel传输到logger sink，由sink将数据写到控制台
```

```
实现过程：
	①在主机hadoop02的配置：
		②安装netcat工具，yum install -y nc
		②进入/opt/flume-1.7.0/conf,拷贝flume-conf.properties.template到/myconf目录,重命名netcat-flume-logger.conf,增加：
			#代理名称是myagent
            #指定服务代理的source、channel和sink有哪些，可以有多个
            myagent.sources = r1
            myagent.channels = c1
            myagent.sinks = k1

            # For each one of the sources, the type is defined
            #配置source的类型，主机，端口
            myagent.sources.r1.type = netcat
            myagent.sources.r1.bind = hadoop02
            myagent.sources.r1.port = 2345

            # The channel can be defined as follows.
            #绑定source与channel，一个source可以对应多个channel
            myagent.sources.r1.channels = c1

            # Each sink's type must be defined
            #配置sink的类型
            myagent.sinks.k1.type = logger

            #Specify the channel the sink should use
            #绑定sink和channel,一个sink只对应一个channel
            myagent.sinks.k1.channel = c1

            # Each channel's type is defined.
            #配置channel的类型,容量1000个event，
            myagent.channels.c1.type = memory
            myagent.channels.c1.capacity = 1000
            myagent.channels.c1.transactionCapacity = 100
     ②在主机hadoop03的配置，安装nc，yum install -y nc
     ③启动：
     	①在hadoop02启动flume服务
     		flume-ng agent -n myagent -f /opt/flume-1.7.0/conf/myconf/netcat-flume-logger.conf -Dflume.root.logger=INFO,console
            注意：
            	①flume-ng命令的格式【-n 代理名 -f 配置文件路径 -Dflume.root.logger=日志级别,日志输出类型】
            	②日志级别：DEBUG,INFO,WARN,ERROR
            	③日志输出类型：LOGFILE(日志文件)，console(控制台)
     	②在hadoop03启动netcat客户端，向hadoop02的2345端口发送数据
     		nc hadoop02 2345
     	③验证：
     		在hadoop02的flume服务接收到来自hadoop03的数据，并打印在控制台
```

##### 案例二：监控指定目录下单个文件

```
需求分析：
	①在hadoop02通过flume source实时监控/opt/hive-1.2.1/logs/hive.log日志文件的变化
	②将变化的内容上传到HDFS
```

```
具体实现：
	①在主机hadoop02的配置：
		①进入/opt/flume-1.7.0/conf,拷贝flume-conf.properties.template到/myconf目录,重命名file-flume-logger.conf,增加：
            #代理名称是myagent
            #指定服务代理的source、channel和sink有哪些，可以有多个
            myagent.sources = r1
            myagent.channels = c1
            myagent.sinks = k1

            #配置source的类型
            myagent.sources.r1.type = exec
            myagent.sources.r1.command = tail -F /opt/hive-1.2.1/logs/hive.log
            myagent.sources.r1.shell = /bin/bash -c

            #绑定source与channel，一个source可以对应多个channel
            myagent.sources.r1.channels = c1

            #########配置sink###########
            myagent.sinks.k1.type = hdfs
            #存放到hdfs上的路径,按天生成目录/按每分钟生成目录/按每10秒生成目录
            myagent.sinks.k1.hdfs.path = hdfs://hadoop02:9000/flume/events/%y-%m-%d/%H%M/%S
            #上传文件的前缀
            myagent.sinks.k1.hdfs.filePrefix = logs-
            #是否按照时间滚动文件夹
            myagent.sinks.k1.hdfs.round = true
            #多少时间创建一个新的文件夹
            myagent.sinks.k1.hdfs.roundValue = 10
            #时间单位，生产中是hour
            myagent.sinks.k1.hdfs.roundUnit = second
            #是否使用本地时间
            myagent.sinks.k1.hdfs.useLocalTimeStamp = true
            #积攒多少个event才flush到HDFS一次
            myagent.sinks.k1.hdfs.batchSize = 1000
            #设置存入HDFS的文件类型，默认是二进制文件
            myagent.sinks.k1.hdfs.fileType = DataStream
            #多久生成一个新文件
            myagent.sinks.k1.hdfs.rollInterval = 2
            #设置文件的滚动大小，生产中与block大小一致
            myagent.sinks.k1.hdfs.rollSize = 134217728
            #文件的滚动与Event的数量无关
            myagent.sinks.k1.hdfs.rollCount = 0

            #绑定sink和channel
            myagent.sinks.k1.channel = c1

            #配置channel的类型,容量1000个event，
            myagent.channels.c1.type = memory
            myagent.channels.c1.capacity = 1000
            myagent.channels.c1.transactionCapacity = 100

	②启动
		①在主机hadoop02启动hdfs服务与hive服务
		②在主机hadoop02启动flume服务
			flume-ng agent -n myagent -f /opt/flume-1.7.0/conf/myconf/file-flume-logger.conf
			注意，从hive.log读取的数据是上传到hdfs，不需要到LOGFILE/console
	③验证
		登录hadoop02:50070,保证hive.log持续有日志追加，然后查看hdfs文件系统/flume目录是否每10秒滚动（生成）一次文件，文件以logs-打头
```

##### 案例三：监控指定目录下多个文件

```
需求分析：
	①在hadoop02通过taildir source实时监控/opt/hive-1.2.1/logs/hive.log文件内容是否发生追加，以及/opt/flume-1.7.0/logs下的是否有txt文件内容发生追加
	②将变化的内容打印到控制台
```

```
具体实现：
	①在主机hadoop02的配置：
		①进入/opt/flume-1.7.0/conf,拷贝flume-conf.properties.template到/myconf目录,重命名files-flume-logger.conf,增加：
		#代理名称是myagent
        #指定服务代理的source、channel和sink有哪些，可以有多个
        myagent.sources = r1
        myagent.channels = c1
        myagent.sinks = k1

        #配置source
        myagent.sources.r1.type = TAILDIR
        #监控文件夹组
        myagent.sources.r1.filegroups = g1 g2
        #指定文件夹组中监控的具体文件
        #监控具体的某个文件
        myagent.sources.r1.filegroups.g1 = /opt/hive-1.2.1/logs/hive.log
        #监控所有的txt文件
        myagent.sources.r1.filegroups.g2 = /opt/flume-1.7.0/logs/.*txt.*
        #记录所有监控的文件的inode信息
        myagent.sources.r1.positionFile = /opt/flume-1.7.0/conf/myconf/position.json

        #绑定source与channel，一个source可以对应多个channel
        myagent.sources.r1.channels = c1

        #配置sink的类型
        myagent.sinks.k1.type = logger

        #绑定sink和channel,一个sink只对应一个channel
        myagent.sinks.k1.channel = c1

        #配置channel的类型,容量1000个event，
        myagent.channels.c1.type = memory
        myagent.channels.c1.capacity = 1000
        myagent.channels.c1.transactionCapacity = 100

    ②启动：
     	①在hadoop02启动flume服务
     		flume-ng agent -n myagent -f /opt/flume-1.7.0/conf/myconf/files-flume-logger.conf -Dflume.root.logger=INFO,console
     	③验证：
     		①启动hive服务，hive.log日志追加内容，查看flume控制台是否打印内容
     		②在目录/opt/flume-1.7.0/logs下创建test.txt文件，追加内容：
     			echo "lsjdlkfjdsf" > test.txt
     			查看flume控制台是否打印内容
```

##### 案例四：监控文件夹

```
需求分析：
	①在hadoop02通过spooling directory source实时监控本地文件夹/opt/flume-1.7.0/logs的变化
	②将变化的内容上传到HDFS
```

```
具体实现：
	①在主机hadoop02的配置：
		①进入/opt/flume-1.7.0/conf,拷贝flume-conf.properties.template到/myconf目录,重命名dir-flume-logger.conf,增加：
			#代理名称是myagent
            #指定服务代理的source、channel和sink有哪些，可以有多个
            myagent.sources = r1
            myagent.channels = c1
            myagent.sinks = k1

            #配置source
            myagent.sources.r1.type = spooldir
            myagent.sources.r1.spoolDir = /opt/flume-1.7.0/logs
            myagent.sources.r1.fileHeader = true
            #只有成功拷贝/上传到spoolDir目录中的文件才会加上后缀，默认是.COMPLETED
            myagent.sources.r1.fileSuffix = .OVER
            #使用正则过滤不需要上传的文件类型，这些文件以原文件名存在（不会加上后缀），且不会被上传到HDFS上
            myagent.sources.r1.ignorePattern = ([^ ]*\.txt)

            #绑定source与channel，一个source可以对应多个channel
            myagent.sources.r1.channels = c1

            #########配置sink###########
            myagent.sinks.k1.type = hdfs
            #存放到hdfs上的路径,按天生成目录/按每分钟生成目录/按每10秒生成目录
            myagent.sinks.k1.hdfs.path = hdfs://hadoop02:9000/flume/events/%y-%m-%d/%H%M/%S
            #上传文件的前缀
            myagent.sinks.k1.hdfs.filePrefix = logs-
            #是否按照时间滚动文件夹
            myagent.sinks.k1.hdfs.round = true
            #多少时间创建一个新的文件夹
            myagent.sinks.k1.hdfs.roundValue = 10
            #时间单位，生产中是hour
            myagent.sinks.k1.hdfs.roundUnit = second
            #是否使用本地时间
            myagent.sinks.k1.hdfs.useLocalTimeStamp = true
            #积攒多少个event才flush到HDFS一次
            myagent.sinks.k1.hdfs.batchSize = 1000
            #设置存入HDFS的文件类型，默认是二进制文件
            myagent.sinks.k1.hdfs.fileType = DataStream
            #多久生成一个新文件
            myagent.sinks.k1.hdfs.rollInterval = 2
            #设置文件的滚动大小，生产中与block大小一致
            myagent.sinks.k1.hdfs.rollSize = 134217728
            #文件的滚动与Event的数量无关
            myagent.sinks.k1.hdfs.rollCount = 0

            #绑定sink和channel
            myagent.sinks.k1.channel = c1

            #配置channel的类型,容量1000个event，
            myagent.channels.c1.type = memory
            myagent.channels.c1.capacity = 1000
            myagent.channels.c1.transactionCapacity = 100
    ②启动
            ①在主机hadoop02启动hdfs服务与hive服务
            ②在主机hadoop02启动flume服务
                flume-ng agent -n myagent -f /opt/flume-1.7.0/conf/myconf/dir-flume-logger.conf
                注意，从hive.log读取的数据是上传到hdfs，不需要到LOGFILE/console
        ③验证
        	①拷贝一个文件到/opt/flume-1.7.0/logs下
            ②登录hadoop02:50070,然后查看hdfs文件系统/flume目录是否每10秒滚动（生成）一次文件，文件以logs-打头
            注意：
            ①flume spooling directory source只能监控到文件数的变化，不能监控到文件内容的变化
            ②不能手动修改/opt/flume-1.7.0/logs下的文件，否则会报错，flume会挂掉
```

##### 案例五：自定义拦截器

```
需求分析：
	①实现事件内容类型拦截器，对
	①在主机hadoop03上通过netcat客户端向服务端2345端口发送数据
	③在主机hadoop02上启动flume监控本机2345端口的数据
	④自定义一个事件内容类型拦截器，实现：
		根据输入内容的类型，将event进行分类
	⑤flume的netcat tcp source组件读取数据，event经过拦截器后打上不同标签，被channel selector以多路复用的方式分发到不同的hdfs sink
```

```
实现过程：
	①在主机hadoop02的配置：
		②安装netcat工具，yum install -y nc
		②进入/opt/flume-1.7.0/conf/myconf,创建配置文件netcat-interceptor-logger.conf,编辑内容：
        #代理名称是myagent
        #指定服务代理的source、channel和sink有哪些，可以有多个
        myagent.sources = r1
        myagent.channels = c1 c2
        myagent.sinks = k1 k2

        #配置source的类型，主机ip/域名，端口
        myagent.sources.r1.type = netcat
        myagent.sources.r1.bind = hadoop02
        myagent.sources.r1.port = 2345

        #配置channel的类型,容量1000个event，
        myagent.channels.c1.type = memory
        myagent.channels.c1.capacity = 1000
        myagent.channels.c1.transactionCapacity = 100

        myagent.channels.c2.type = memory
        myagent.channels.c2.capacity = 1000
        myagent.channels.c2.transactionCapacity = 100

        #interceptor
        myagent.sources.r1.interceptors = i1
        #消息分类拦截器
        myagent.sources.r1.interceptors.i1.type = flume.SimpleInterceptor$Builder

        #channel selector
        #选择器的类型有两种：一种是multiplexing（多路复用），根据消息事件event的header中定义的（key,value）将内容上传到hdfs的不同根目录；另一种是replicating（复本，默认这种），会发送给所有的sink
        myagent.sources.r1.selector.type = multiplexing
        #指定header中自定义的标签（在自定义日志类型拦截器SimpleInterceptor中，已经为每个消息事件event打上了标签("type","num")、("type","alpha")）
        myagent.sources.r1.selector.header = type
        #绑定value与channel映射关系
        myagent.sources.r1.selector.mapping.num = c1
        myagent.sources.r1.selector.mapping.alpha = c2

        #配置sink的类型
        myagent.sinks.k1.type = hdfs
        #存放到hdfs上的路径,按天生成目录/按每分钟生成目录/按每10秒生成目录
        myagent.sinks.k1.hdfs.path = hdfs://hadoop02:9000/flume/events/num/%y-%m-%d/%H%M/%S
        #上传文件的前缀
        myagent.sinks.k1.hdfs.filePrefix = logs-
        #是否按照时间滚动文件夹
        myagent.sinks.k1.hdfs.round = true
        #多少时间创建一个新的文件夹
        myagent.sinks.k1.hdfs.roundValue = 10
        #时间单位，生产中是hour
        myagent.sinks.k1.hdfs.roundUnit = second
        #是否使用本地时间
        myagent.sinks.k1.hdfs.useLocalTimeStamp = true
        #积攒多少个event才flush到HDFS一次
        myagent.sinks.k1.hdfs.batchSize = 1000
        #设置存入HDFS的文件类型，默认是二进制文件
        myagent.sinks.k1.hdfs.fileType = DataStream
        #多久生成一个新文件
        myagent.sinks.k1.hdfs.rollInterval = 2
        #设置文件的滚动大小，生产中与block大小一致
        myagent.sinks.k1.hdfs.rollSize = 134217728
        #文件的滚动与Event的数量无关
        myagent.sinks.k1.hdfs.rollCount = 0

        myagent.sinks.k2.type = hdfs
        #存放到hdfs上的路径,按天生成目录/按每分钟生成目录/按每10秒生成目录
        myagent.sinks.k2.hdfs.path = hdfs://hadoop02:9000/flume/events/alpha/%y-%m-%d/%H%M/%S
        #上传文件的前缀
        myagent.sinks.k2.hdfs.filePrefix = logs-
        #是否按照时间滚动文件夹
        myagent.sinks.k2.hdfs.round = true
        #多少时间创建一个新的文件夹
        myagent.sinks.k2.hdfs.roundValue = 10
        #时间单位，生产中是hour
        myagent.sinks.k2.hdfs.roundUnit = second
        #是否使用本地时间
        myagent.sinks.k2.hdfs.useLocalTimeStamp = true
        #积攒多少个event才flush到HDFS一次
        myagent.sinks.k2.hdfs.batchSize = 1000
        #设置存入HDFS的文件类型，默认是二进制文件
        myagent.sinks.k2.hdfs.fileType = DataStream
        #多久生成一个新文件
        myagent.sinks.k2.hdfs.rollInterval = 2
        #设置文件的滚动大小，生产中与block大小一致
        myagent.sinks.k2.hdfs.rollSize = 134217728
        #文件的滚动与Event的数量无关
        myagent.sinks.k2.hdfs.rollCount = 0

        #绑定source与channel，一个source可以对应多个channel
        myagent.sources.r1.channels = c1 c2
        #绑定sink和channel,一个sink只对应一个channel
        myagent.sinks.k1.channel = c1
        myagent.sinks.k2.channel = c2
     ③将big-data工程下的模块big-data-04-flume打包成jar，将big-data-04-flume-1.0-SNAPSHOT.jar文件上传到主机hadoop02 linux文件系统/opt/flume-1.7.0/lib/下
     ④验证：
     	①在hadoop02，运行flume-ng客户端，使用命令：
     		flume-ng agent -n myagent -f /opt/flume-1.7.0/conf/myconf/netcat-interceptor-logger.conf
     	②在hadoop03启动netcat客户端，向hadoop02的2345端口发送数据
     		nc hadoop02 2345
     	③登录hadoop02:50070，查看hdfs://hadoop02:9000/flume/events/下是否滚动生成日志文件
```

##### 案例总结

| 案例   | source            | 原理                                                         | 数据是否会丢失                        | 断点续传 | 实时性 |
| ------ | ----------------- | ------------------------------------------------------------ | ------------------------------------- | -------- | ------ |
| 案例一 | tcp netcat source | netcat工具实现tcp通信，需要提前安装netcat工具                | 不丢失                                | 否       | 实时   |
| 案例二 | exec source       | 通过shell工具执行myagent.sources.r1.command中的命令实现对文件内容的监控 | 由于flume agent存在单点故障而丢失数据 | 否       | 实时   |
| 案例三 | spooldir source   | **避免数据丢失**：对上传成功的文件名增加后缀，下次会忽略这些有后缀名的，只上传不含后缀名的文件； | 不丢失                                | 否       | 非实时 |
| 案例四 | taildir source    | **断点续传原理**：指定一个json文件记录每次上传完成时读文件的位置（inode），如果发生单点故障，下次重新从记录的位置接着上传；**避免数据丢失**：json中记录着inode唯一标识这个文件，如果文件被修改/移动，新的inode与json中的inode不一致，agent将重新上传这个文件 | 不丢失                                | 是       | 实时   |

### Flume企业开发案例

##### 案例一：复制和多路复用

```
需求分析：
	①需要多个Flume agent串联完成
	②Flume1负责监控/opt/flume-1.7.0/cases/case1/case1.log内容变动，将变动内容传递给Flume2和Flume3
	③Flume2负责将Flume1的内容存储到HDFS
	④Flume3负责将Flume1的内容输出到本地文件系统
```

```
具体实现：
	①准备工作：在主机hadoop02上
		①创建/opt/flume-1.7.0/cases/case1目录，添加
            flume1的配置文件：flume1.conf
            flume2的配置文件：flume2.conf
            flume3的配置文件：flume3.conf
            flume1的断点续传用于记录inode的json文件：position.json
            flume1监控的文件：case1.log
         ②创建/opt/flume-1.7.0/cases/case1/output目录,用于flume3存放监控结果
         	
    ②conf配置文件：
    	①编辑flume1.conf,添加：
    		#监听一个文件只需要一个source
            #一个文件需要复制到多个sink，因为sink不支持复制，因此也需要多个channel
            #k1传给flume2，k2传给flume3
            myagent1.sources = r1
            myagent1.channels = c1 c2
            myagent1.sinks = k1 k2

            #source 
            myagent1.sources.r1.type = TAILDIR
            myagent1.sources.r1.filegroups = g1
            myagent1.sources.r1.filegroups.g1 = /opt/flume-1.7.0/cases/case1/case1.log
            myagent1.sources.r1.positionFile = /opt/flume-1.7.0/cases/case1/position.json

            #channel c1
            myagent1.channels.c1.type = memory
            myagent1.channels.c1.capacity = 1000
            myagent1.channels.c1.transactionCapacity = 100

            #channel c2
            myagent1.channels.c2.type = memory
            myagent1.channels.c2.capacity = 1000
            myagent1.channels.c2.transactionCapacity = 100

            #sink k1
            myagent1.sinks.k1.type = avro
            myagent1.sinks.k1.hostname = hadoop02
            myagent1.sinks.k1.port = 3456

            #sink k2
            myagent1.sinks.k2.type = avro
            myagent1.sinks.k2.hostname = hadoop02
            myagent1.sinks.k2.port = 4567

            #绑定source和channel，sink和channel
            myagent1.sources.r1.channels = c1 c2
            myagent1.fsinks.k1.channel = c1
            myagent1.sinks.k2.channel = c2
            
         ①编辑flume2.conf,添加：
         	#Name
            myagent2.sources = r1
            myagent2.channels = c1
            myagent2.sinks = k1

            #source
            myagent2.sources.r1.type = avro
            myagent2.sources.r1.bind = hadoop02
            myagent2.sources.r1.port = 3456

            #channel
            myagent2.channels.c1.type = memory
            myagent2.channels.c1.capacity = 1000
            myagent2.channels.c1.transactionCapacity = 100

            #sink 
            myagent2.sinks.k1.type = hdfs
            #存放到hdfs上的路径,按天生成目录/按每分钟生成目录/按每10秒生成目录
            myagent2.sinks.k1.hdfs.path = hdfs://hadoop02:9000/flume/case1/events/%y-%m-%d/%H%M/%S
            #上传文件的前缀
            myagent2.sinks.k1.hdfs.filePrefix = logs-
            #是否按照时间滚动文件夹
            myagent2.sinks.k1.hdfs.round = true
            #多少时间创建一个新的文件夹
            myagent2.sinks.k1.hdfs.roundValue = 10
            #时间单位，生产中是hour
            myagent2.sinks.k1.hdfs.roundUnit = second
            #是否使用本地时间
            myagent2.sinks.k1.hdfs.useLocalTimeStamp = true
            #积攒多少个event才flush到HDFS一次
            myagent2.sinks.k1.hdfs.batchSize = 1000
            #设置存入HDFS的文件类型，默认是二进制文件
            myagent2.sinks.k1.hdfs.fileType = DataStream
            #多久生成一个新文件
            myagent2.sinks.k1.hdfs.rollInterval = 2
            #设置文件的滚动大小，生产中与block大小一致
            myagent2.sinks.k1.hdfs.rollSize = 134217728
            #文件的滚动与Event的数量无关
            myagent2.sinks.k1.hdfs.rollCount = 0

            #bind
            myagent2.sources.r1.channels = c1
            myagent2.sinks.k1.channel = c1
            
        ③编辑flume3.conf,添加：
        	#Name
            myagent3.sources = r1
            myagent3.channels = c1
            myagent3.sinks = k1

            #source
            myagent3.sources.r1.type = avro
            myagent3.sources.r1.bind = hadoop02
            myagent3.sources.r1.port = 4567

            #channel
            myagent3.channels.c1.type = memory
            myagent3.channels.c1.capacity = 1000
            myagent3.channels.c1.transactionCapacity = 100

            #sink 
            myagent3.sinks.k1.type = file_roll
            myagent3.sinks.k1.sink.directory = /opt/flume-1.7.0/cases/case1/output

            #bind
            myagent3.sources.r1.channels = c1
            myagent3.sinks.k1.channel = c1
    ③启动：
    	先启动flume2和flume3，再启动flume1，分别使用命令：
    		flume-ng agent -n myagent2 -f /opt/flume-1.7.0/cases/case1/flume2.conf
    		flume-ng agent -n myagent3 -f /opt/flume-1.7.0/cases/case1/flume3.conf
    		flume-ng agent -n myagent1 -f /opt/flume-1.7.0/cases/case1/flume1.conf
    ④验证：
    	向/opt/flume-1.7.0/cases/case1/case1.log追加内容
    	①观察/opt/flume-1.7.0/cases/case1/output路径下是否生成日志
    	②观察文件系统hdfs://hadoop02:9000/flume/case1/events/是否生成日志
```

##### 案例二：sink组故障转移

```
需求分析：
	①在主机hadoop03上通过netcat客户端向服务端2345端口发送数据
	②在主机hadoop02上启动flume1监控本机2345端口的数据
	②flume1将数据转发给sink group，sink组中的sink分别对接flume2和flume3
	④sink group按照故障转移策略将数据转发给flume2或者flume3，由flume2或者flume3将结果打印到控制台
```

```
具体实现：
	①准备工作：在主机hadoop02上
		①创建/opt/flume-1.7.0/cases/case2目录，添加
            flume1的配置文件：flume1.conf
            flume2的配置文件：flume2.conf
            flume3的配置文件：flume3.conf
         	
    ②conf配置文件：
    	①编辑flume1.conf,添加：
    		#Name
            myagent1.sources = r1
            myagent1.channels = c1
            myagent1.sinks = k1 k2

            #source
            myagent1.sources.r1.type = netcat
            myagent1.sources.r1.bind = hadoop02
            myagent1.sources.r1.port = 2345

            #channel
            myagent1.channels.c1.type = memory
            myagent1.channels.c1.capacity = 1000
            myagent1.channels.c1.transactionCapacity = 100

            #sink k1
            myagent1.sinks.k1.type = avro
            myagent1.sinks.k1.hostname = hadoop02
            myagent1.sinks.k1.port = 3456

            #sink k2
            myagent1.sinks.k2.type = avro
            myagent1.sinks.k2.hostname = hadoop02
            myagent1.sinks.k2.port = 4567

            #sink group
            #k1 k2一组
            myagent1.sinkgroups.g1.sinks = k1 k2
            #sink组的类型
            myagent1.sinkgroups.g1.processor.type = failover
            #sink的优先级，值大的优先
            myagent1.sinkgroups.g1.processor.priority.k1 = 5
            myagent1.sinkgroups.g1.processor.priority.k2 = 50
            #10秒，sink复活时间，如果k2挂了，agent将数据转发给k1，超过10秒后k2复活了，才会继续优先转发给k2，如果10秒内k2就复活了，也不会使用k2 
            myagent1.sinkgroups.g1.processor.maxpenalty = 10000

            #绑定
            myagent1.sources.r1.channels = c1
            myagent1.sinks.k1.channel = c1
            myagent1.sinks.k2.channel = c1
            
		②编辑flume2.conf,添加：
			#Name
            myagent2.sources = r1
            myagent2.channels = c1
            myagent2.sinks = k1

            #source
            myagent2.sources.r1.type = avro
            myagent2.sources.r1.bind = hadoop02
            myagent2.sources.r1.port = 3456

            #channel
            myagent2.channels.c1.type = memory
            myagent2.channels.c1.capacity = 1000
            myagent2.channels.c1.transactionCapacity = 100

            #sink k1
            myagent2.sinks.k1.type = logger

            #绑定
            myagent2.sources.r1.channels = c1
            myagent2.sinks.k1.channel = c1
            
		③编辑flume3.conf,添加：
			#Name
            myagent3.sources = r1
            myagent3.channels = c1
            myagent3.sinks = k1

            #source
            myagent3.sources.r1.type = avro
            myagent3.sources.r1.bind = hadoop02
            myagent3.sources.r1.port = 4567

            #channel
            myagent3.channels.c1.type = memory
            myagent3.channels.c1.capacity = 1000
            myagent3.channels.c1.transactionCapacity = 100

            #sink k1
            myagent3.sinks.k1.type = logger

            #绑定
            myagent3.sources.r1.channels = c1
            myagent3.sinks.k1.channel = c1
	③启动：
    	先启动flume2和flume3，再启动flume1，分别使用命令：
    		flume-ng agent -n myagent2 -f /opt/flume-1.7.0/cases/case2/flume2.conf
    		flume-ng agent -n myagent3 -f /opt/flume-1.7.0/cases/case2/flume3.conf
    		flume-ng agent -n myagent1 -f /opt/flume-1.7.0/cases/case2/flume1.conf
    ④验证：
    	在hadoop03主机开启netcat客户端，nc hadoop02 2345，并发送数据
    	①由于sink组按照故障转移机制的权重优先策略将数据转发给flume3，观察flume3是否有数据
    	②关闭flume3服务，数据则发给flume2
```

##### 案例三：sink组负载均衡

```
需求分析：
	①在主机hadoop03上通过netcat客户端向服务端2345端口发送数据
	②在主机hadoop02上启动flume1监控本机2345端口的数据
	②flume1将数据转发给sink group，sink组中的sink分别对接flume2和flume3
	④sink group按照负载均衡机制将数据转发给flume2或者flume3，由flume2或者flume3将结果打印到控制台
```

```
具体实现：
	①准备工作：在主机hadoop02上
		①创建/opt/flume-1.7.0/cases/case3目录，添加
            flume1的配置文件：flume1.conf
            flume2的配置文件：flume2.conf
            flume3的配置文件：flume3.conf
         	
    ②conf配置文件：
    	①编辑flume1.conf,添加：
    		#Name
            myagent1.sources = r1
            myagent1.channels = c1
            myagent1.sinks = k1 k2

            #source
            myagent1.sources.r1.type = netcat
            myagent1.sources.r1.bind = hadoop02
            myagent1.sources.r1.port = 2345

            #channel
            myagent1.channels.c1.type = memory
            myagent1.channels.c1.capacity = 1000
            myagent1.channels.c1.transactionCapacity = 100

            #sink k1
            myagent1.sinks.k1.type = avro
            myagent1.sinks.k1.hostname = hadoop02
            myagent1.sinks.k1.port = 3456

            #sink k2
            myagent1.sinks.k2.type = avro
            myagent1.sinks.k2.hostname = hadoop02
            myagent1.sinks.k2.port = 4567

            #sink group
            #k1 k2一组
            myagent1.sinkgroups.g1.sinks = k1 k2
            #sink组的类型
            a1.sinkgroups.g1.processor.type = load_balance
            #失败的sink，开启回退
            a1.sinkgroups.g1.processor.backoff = true
            #选择策略，random（随机选择）/round_robin（轮询）
            a1.sinkgroups.g1.processor.selector = random

            #绑定
            myagent1.sources.r1.channels = c1
            myagent1.sinks.k1.channel = c1
            myagent1.sinks.k2.channel = c1
            
		②编辑flume2.conf,添加：
			#Name
            myagent2.sources = r1
            myagent2.channels = c1
            myagent2.sinks = k1

            #source
            myagent2.sources.r1.type = avro
            myagent2.sources.r1.bind = hadoop02
            myagent2.sources.r1.port = 3456

            #channel
            myagent2.channels.c1.type = memory
            myagent2.channels.c1.capacity = 1000
            myagent2.channels.c1.transactionCapacity = 100

            #sink k1
            myagent2.sinks.k1.type = logger

            #绑定
            myagent2.sources.r1.channels = c1
            myagent2.sinks.k1.channel = c1
            
		③编辑flume3.conf,添加：
			#Name
            myagent3.sources = r1
            myagent3.channels = c1
            myagent3.sinks = k1

            #source
            myagent3.sources.r1.type = avro
            myagent3.sources.r1.bind = hadoop02
            myagent3.sources.r1.port = 4567

            #channel
            myagent3.channels.c1.type = memory
            myagent3.channels.c1.capacity = 1000
            myagent3.channels.c1.transactionCapacity = 100

            #sink k1
            myagent3.sinks.k1.type = logger

            #绑定
            myagent3.sources.r1.channels = c1
            myagent3.sinks.k1.channel = c1
	③启动：
    	先启动flume2和flume3，再启动flume1，分别使用命令：
    		flume-ng agent -n myagent2 -f /opt/flume-1.7.0/cases/case2/flume2.conf
    		flume-ng agent -n myagent3 -f /opt/flume-1.7.0/cases/case2/flume3.conf
    		flume-ng agent -n myagent1 -f /opt/flume-1.7.0/cases/case2/flume1.conf
    ④验证：
    	在hadoop03主机开启netcat客户端，nc hadoop02 2345，并发送数据
    	①由于采用sink组负载均衡的随机选择策略，数据可能被转发给flume2或者flume3
    	②如果关闭flume2服务，数据则只会发给flume3
```

##### 案例四：聚合

```
需求分析：
	①hadoop02上的flume1监控文件/opt/flume-1.7.0/cases/case4/case4.log文件的变化
	②hadoop03上的flume2监控端口2345的数据流
	③flume1和flume2将数据发送给hadoop04上的flume3
	④flume3将数据打印到控制台
```

```
具体实现：
	①准备工作：
		①在主机hadoop02上，创建/opt/flume-1.7.0/cases/case4目录，添加
            flume1的配置文件：flume1.conf
			flume1的断点续传用于记录inode的json文件：position.json
            flume1监控的文件：case4.log
        ②在主机hadoop03上，创建/opt/flume-1.7.0/cases/case4目录，添加
        	flume2的配置文件：flume2.conf
        ②在主机hadoop04上，创建/opt/flume-1.7.0/cases/case4目录，添加
        	flume3的配置文件：flume3.conf
    ②配置文件：
    	①编辑flume1.conf,添加：
    		#Name
            myagent1.sources = r1
            myagent1.channels = c1
            myagent1.sinks = k1

            #source
            myagent1.sources.r1.type = TAILDIR
            #监控文件夹组
            myagent1.sources.r1.filegroups = g1
            #指定文件夹组中监控的具体文件
            #监控具体的某个文件
            myagent1.sources.r1.filegroups.g1 = /opt/flume-1.7.0/cases/case4/case4.log
            #记录所有监控的文件的inode信息
            myagent1.sources.r1.positionFile = /opt/flume-1.7.0/cases/case4/position.json

            #channel
            myagent1.channels.c1.type = memory
            myagent1.channels.c1.capacity = 1000
            myagent1.channels.c1.transactionCapacity = 100

            #sink
            myagent1.sinks.k1.type = avro
            myagent1.sinks.k1.hostname = hadoop04
            myagent1.sinks.k1.port = 3456

            #绑定
            myagent1.sources.r1.channels = c1
            myagent1.sinks.k1.channel = c1

    	②编辑flume2.conf,添加：
    		#Name
            myagent2.sources = r1
            myagent2.channels = c1
            myagent2.sinks = k1

            #source
            myagent2.sources.r1.type = netcat
            myagent2.sources.r1.bind = hadoop03
            myagent2.sources.r1.port = 2345

            #channel
            myagent2.channels.c1.type = memory
            myagent2.channels.c1.capacity = 1000
            myagent2.channels.c1.transactionCapacity = 100

            #sink
            myagent2.sinks.k1.type = avro
            myagent2.sinks.k1.hostname = hadoop04
            myagent2.sinks.k1.port = 3456

            #绑定
            myagent2.sources.r1.channels = c1
            myagent2.sinks.k1.channel = c1
            
    	③编辑flume3.conf,添加：
    		#Name
            myagent3.sources = r1
            myagent3.channels = c1
            myagent3.sinks = k1

            #source
            myagent3.sources.r1.type = avro
            myagent3.sources.r1.bind = hadoop04
            myagent3.sources.r1.port = 3456

            #sink
            myagent3.sinks.k1.type = logger

            #channel
            myagent3.channels.c1.type = memory
            myagent3.channels.c1.capacity = 1000
            myagent3.channels.c1.transactionCapacity = 100

            #绑定
            myagent3.sources.r1.channels = c1
            myagent3.sinks.k1.channel = c1
            
    ③启动：
    	先启动flume3，再启动flume1和flume2，分别使用命令：
    		flume-ng agent -n myagent3 -f /opt/flume-1.7.0/cases/case4/flume3.conf
    		flume-ng agent -n myagent1 -f /opt/flume-1.7.0/cases/case4/flume1.conf
    		flume-ng agent -n myagent2 -f /opt/flume-1.7.0/cases/case4/flume2.conf
    ④验证：
    	①在hadoop02主机开启netcat客户端，nc hadoop03 2345，向hadoop03发送数据，观察flume3控制台
    	①在hadoop02主机向/opt/flume-1.7.0/cases/case4/case4.log追加内容，观察flume3控制台
```

##### 案例五：对接kafka

##### kafka在大数据中的应用

```
kafka作为消息中间件，实现flume与计算框架之间的解耦，生产中使用flume+kafka的业务场景：
	第一：flume将日志数据写入kafka，kafka再将数据写入hdfs
		分析flume不直接写到hdfs的原因：
			①实时性：hdfs的实时性没有kafka高。
	第二：计算框架（spark、flink）从kafka获取数据进行实时计算
		分析计算框架不直接与hdfs交互的原因：
			①消费量的记录：hdfs不会记录你这个块文件消费到了哪里，而基于zookeeper的kafka会记录你消费的点。
			②并发消费：hdfs不支持并发消费，而kafka支持并发消费，即多个consumer.
			③弹性且有序：当数据量会很大，而且处理完之后就可以删除时，频繁的读写会对hdfs中NameNode造成很大的压力。而kafka的消费点是记录在zookeeper的，并且kafka的每条数据都是有“坐标”的，所以消费的时候只要这个“坐标”向后移动就行了，而且删除的时候只要把这个“坐标”之前的数据删掉即可
```

##### file-flume-kafka

```
需求分析：
	①实时监控hadoop02文件系统目录/opt/project_data_warehouse/logs的变化
	②自定义一个日志分类拦截器，实现：
		根据输入内容的类型，将event进行分类
	③自定义一个日志清洗拦截器，实现：
		筛选掉不符合格式要求的event
	④flume的taildir source组件读取数据，event经过拦截器后打上不同标签，被channel selector以多路复用的方式分发到kafka的不同主题
```

```
具体实现：
	①准备工作：
		①在主机hadoop02上，创建/opt/flume-1.7.0/cases/case5目录，添加
            配置文件：file-flume-kafka.conf
			断点续传用于记录inode的json文件：position.json
	②具体配置：
		①编辑file-flume-kafka.conf文件：
            #Name
            myagent.channels = c1 c2
            myagent.sources = r1

            #source
            myagent.sources.r1.type = TAILDIR
            #监控文件夹组
            myagent.sources.r1.filegroups = g1
            #指定文件夹组中监控的具体文件
            #监控具体的app打头的文件
            myagent.sources.r1.filegroups.g1 = /opt/project_data_warehouse/logs/app.+
            #记录所有监控的文件的inode信息
            myagent.sources.r1.positionFile = /opt/flume-1.7.0/cases/case5/position.json
            myagent.sources.r1.fileHeader = true

            #interceptor
            myagent.sources.r1.interceptors = i1 i2
            #日志数据清洗拦截器
            myagent.sources.r1.interceptors.i1.type = flume.LogETLInterceptor$Builder
            myagent.sources.r1.interceptors.i1.preserveExisting = false
            #日志分类拦截器
            myagent.sources.r1.interceptors.i2.type = flume.LogTypeInterceptor$Builder
            myagent.sources.r1.interceptors.i2.preserveExisting = false

            #channel selector
            #选择器的类型有两种：一种是multiplexing（多路复用），根据消息事件event的header中定义的（key,value）将内容上传到hdfs的不同根目录；另一种是replicating（复本，默认这种），会发送给所有的sink
            myagent.sources.r1.selector.type = multiplexing
            #指定header中自定义的标签（在自定义日志类型拦截器LogTypeInterceptor中，已经为每个消息事件event打上了标签("topic","topic_start")/("topic","topic_event")）
            myagent.sources.r1.selector.header = topic
            #绑定value与channel映射关系
            myagent.sources.r1.selector.mapping.topic_start = c1
            myagent.sources.r1.selector.mapping.topic_event = c2

            #channel
            myagent.channels.c1.type = org.apache.flume.channel.kafka.KafkaChannel
            #kafka集群地址
            myagent.channels.c1.kafka.bootstrap.servers = hadoop02:9092,hadoop03:9092,hadoop04:9092
            #主题
            myagent.channels.c1.kafka.topic = topic_start
            #是否将body和header拼接到一个消息中
            myagent.channels.c1.parseAsFlumeEvent = false
            #多个channel公用的消费者组ID，flume通过这个ID向kafka注册，将event写入kafka主题topic_start
            myagent.channels.c1.kafka.consumer.group.id = flume-consumer

            myagent.channels.c2.type = org.apache.flume.channel.kafka.KafkaChannel
            myagent.channels.c2.kafka.bootstrap.servers = hadoop02:9092,hadoop03:9092,hadoop04:9092
            #主题
            myagent.channels.c2.kafka.topic = topic_event
            #是否将body和header拼接到一个消息中
            myagent.channels.c2.parseAsFlumeEvent = false
            #多个channel公用的消费者组ID，flume通过这个ID向kafka注册，向kafka生产消息
            myagent.channels.c2.kafka.consumer.group.id = flume-consumer

            #sink
            #kafka channel直接将event写入kafka主题，不需要sink
            #flume的一个agent包括source、channel、sink，但是有时候其实可以不需要source或者是sink，比如在使用flume时，把flume监控的日志发送到kafka中，可以不用使用sink，把channel定义为kafkachannel，kafkachannel作为生产者，直接把数据生产到kafka中，或者使用flume监控kafka中的数据，把数据发送到其他地方，可以不使用source，使用kafkachannel作为消费者，从kafka中拉数据，把数据写到指定的sink上

            #bind
            myagent.sources.r1.channels = c1 c2

        ②将IDEA中的工程big-data-project-Ecommerce-data-warehouse下的模块project-01-log-collector打包成jar，将project-01-log-collector-1.0-SNAPSHOT-jar-with-dependencies.jar上传到hadoop02的linux文件系统目录/opt/project_data_warehouse下
        ③将IDEA中的工程big-data-project-Ecommerce-data-warehouse下的模块project-01-flume-interceptor打包成jar，将project-01-flume-interceptor-1.0-SNAPSHOT.jar上传到/opt/flume-1.7.0/lib目录下
     ③验证：
     	①启动kafka集群，使用自定义批量群起命令：
     		kafka-batch-start
     	②启动flume-ng客户端，使用命令：
     		flume-ng agent -n myagent -c /opt/flume-1.7.0/cases/case5 -f /opt/flume-1.7.0/cases/case5/file-flume-kafka.conf -Dflume.root.logger=DEBUG,LOGFILE
     	③进入目录/opt/project_data_warehouse，运行命令：
        	java -classpath project-01-log-collector-1.0-SNAPSHOT-jar-with-dependencies.jar project.AppMain
        ④使用zkCli.sh登录zookeeper客户端，查看/brokers/topics/下是否生成主题topic_start和主题topic_event
```

##### kafka-flume-hdfs

```
需求分析：
	①使用kafka source将kafka中的主题topic_start和topic_event中的数据分别上传到file channel的本地目录/opt/flume-1.7.0/cases/case5/data/start_log、/opt/flume-1.7.0/cases/case5/data/event_log
	②使用file channel将/start_log和/event_log中的数据分别上传到hdfs://hadoop02:9000/usr/project_data_warehouse/logs/topic_start目录、hdfs://hadoop02:9000/usr/project_data_warehouse/logs/topic_event目录
```

```
具体实现：
	①在主机hadoop02，目录/opt/flume-1.7.0/cases/case5/下创建kafka-flume-hdfs.conf
	②编辑kafka-flume-hdfs.conf：
		#Name
        #topic-start和topic-event两个主题的数据源
        myagent1.sources = r1 r2
        #source与channel一一对应
        myagent1.channels = c1 c2
        #写到hdfs的不同路径下
        myagent1.sinks = k1 k2

        #source
        myagent1.sources.r1.type = org.apache.flume.source.kafka.KafkaSource
        #一次批处理最多包含的消息数,如果event大小1K，batchSize建议设成500-1000
        myagent1.sources.r1.batchSize = 1000
        #批写入source的最大时间（毫秒）
        myagent1.sources.r1.batchDurationMillis = 2000
        #数据源的地址
        myagent1.sources.r1.kafka.bootstrap.servers = hadoop02:9092,hadoop03:9092,hadoop04:9092
        #要消费的主题
        myagent1.sources.r1.kafka.topics = topic_start

        myagent1.sources.r2.type = org.apache.flume.source.kafka.KafkaSource
        #一次批处理最多包含的消息数,如果event大小1K，batchSize建议设成500-1000
        myagent1.sources.r2.batchSize = 1000
        #批写入source的最大时间（毫秒）
        myagent1.sources.r2.batchDurationMillis = 2000
        #数据源的地址
        myagent1.sources.r2.kafka.bootstrap.servers = hadoop02:9092,hadoop03:9092,hadoop04:9092
        #要消费的主题
        myagent1.sources.r2.kafka.topics = topic_event

        #channel
        ###########这里如果直接使用flume 的kafka channel（直接作为消费者从kafka拉取数据），可以不需要配置source######
        #######这里为了保证数据绝对安全，考虑使用file channel，先将kafka数据拉取到文件系统#########
        myagent1.channels.c1.type = file
        #检查点存放目录
        myagent1.channels.c1.checkpointDir = /opt/flume-1.7.0/cases/case5/checkpoint/checkpoint1
        #存放启动日志的目录
        myagent1.channels.c1.dataDirs = /opt/flume-1.7.0/cases/case5/data/start_log
        #单个日志文件的最大字节数（接近2G）
        myagent1.channels.c1.maxFileSize = 2146435071
        #channel存放event指针对象的最大容量
        myagent1.channels.c1.capacity = 100000
        myagent1.channels.c1.keep-alive = 6

        myagent1.channels.c2.type = file
        myagent1.channels.c2.checkpointDir = /opt/flume-1.7.0/cases/case5/checkpoint/checkpoint2
        #存放事件日志的目录
        myagent1.channels.c2.dataDirs = /opt/flume-1.7.0/cases/case5/data/event_log
        #单个日志文件的最大字节数（接近2G）
        myagent1.channels.c2.maxFileSize = 2146435071
        #channel存放event指针对象的最大容量
        myagent1.channels.c2.capacity = 100000
        myagent1.channels.c1.keep-alive = 6

        #sink
        myagent1.sinks.k1.type = hdfs
        #hdfs存放目录
        myagent1.sinks.k1.hdfs.path = /usr/project_data_warehouse/logs/topic_start/%Y-%m-%d
        #日志文件的前缀
        myagent1.sinks.k1.hdfs.filePrefix = logstart-

        myagent1.sinks.k2.type = hdfs
        #hdfs存放目录
        myagent1.sinks.k2.hdfs.path = /usr/project_data_warehouse/logs/topic_event/%Y-%m-%d
        #日志文件的前缀
        myagent1.sinks.k2.hdfs.filePrefix = logevent-

        #避免产生大量小文件
        #默认每隔10秒滚动（生成）一个新的日志文件,生产中建议配成3600（一小时）
        myagent1.sinks.k1.hdfs.rollInterval = 10
        #如果日志文件大小超过1024字节，也会触发滚动，生产中建议设置成HDFS块大小（128M）
        myagent1.sinks.k1.hdfs.rollSize = 134217728
        #如果event的数量达到这个值也会触发滚动，设成0表示event的数量不再触发滚动
        myagent1.sinks.k1.hdfs.rollCount = 0

        #默认每隔10秒滚动（生成）一个新的日志文件,生产中建议配成3600（一小时）
        myagent1.sinks.k2.hdfs.rollInterval = 10
        #如果日志文件大小超过1024字节，也会触发滚动，生产中建议设置成HDFS块大小（128M）
        myagent1.sinks.k2.hdfs.rollSize = 134217728
        #如果event的数量达到这个值也会触发滚动，设成0表示event的数量不再触发滚动
        myagent1.sinks.k2.hdfs.rollCount = 0

        #日志压缩存储
        #开启压缩
        myagent1.sinks.k1.hdfs.fileType = CompressedStream
        myagent1.sinks.k2.hdfs.fileType = CompressedStream

        myagent1.sinks.k1.hdfs.codeC = lzop
        myagent1.sinks.k2.hdfs.codeC = lzop

        #绑定
        myagent1.sources.r1.channels = c1
        myagent1.sources.r2.channels = c2
        myagent1.sinks.k1.channel = c1
        myagent1.sinks.k2.channel = c2
    ③验证：
    	①启动kafka集群，使用命令：
    		kafka-batch-start
    	②启动flume-ng客户端，使用命令：
    		flume-ng agent -n myagent1 -f /opt/flume-1.7.0/cases/case5/kafka-flume-hdfs.conf
```

### Flume监控之Ganglia

##### Ganglia简介

```
Ganglia是跨平台，可扩展的分布式监控系统。可同时监控系统数以千计的节点的性能，如cpu、内存、磁盘、i/o
```

##### 安装依赖

```
依次安装：
①安装httpd服务和php
	install -y install httpd php
②安装其它依赖
	yum -y install rrdtool perl-rrdtool rrdtool-devel
	yum -y install epel-release
③安装ganglia的组件
	yum install -y ganglia-gmetad
	yum install -y ganglia-web
	yum install -y ganglia-gmond
注意：根据linux的版本的不同所安装的依赖也不相同，centos 7 mininal版本还需要安装mysql-compat
```

##### 配置

```
①修改/etc/httpd/conf.d/ganglia.conf
	Require all granted
②修改/etc/ganglia/gmetad.conf
	data_source "hadoop02" 192.168.8.105
③修改/etc/ganglia/gmond.conf
	cluster {name = "hadoop02"}
	udp_send_channel {host = 192.168.8.105}
	udp_recv_channel {bind = 192.168.8.105}
④修改/etc/selinux/config
	SELINUX=disabled
重启系统，使配置生效
```

##### 使用

```
①启动服务
	service httpd start
	service gmetad start
	service gmond start
②启动flume任务
	①在主机hadoop02，首先开启hdfs集群和yarn集群
	②指定flume的监控是ganglia
		两种方式：
		①配置flume-env.sh,使用ganglia来监控
            JAVA_OPTS="-Dflume.monitoring.type=ganglia    //指定监控方式为ganglia
            -Dflume.monitoring.hosts=192.168.8.105:8649   //指定ganglia服务的地址
            -Xms100m 	//限定flume服务启动占用的jvm堆大小
            -Xmx200m"	//最大堆
		②启动flume的时候增加参数
			flume-ng agent -n myagent -f /opt/flume-1.7.0/conf/myconf/netcat-flume-logger.conf -Dflume.monitoring.type=ganglia -Dflume.monitoring.hosts=192.168.8.105:8649
			-Xms100m
			-Xmx200m
③访问192.168.8.105/ganglia
```

### Flume多层拓扑

```
由于flume本身不支持分布式，为了保证flume服务的高可用和避免单点故障，需要将flume部署成多层拓扑结构
```

