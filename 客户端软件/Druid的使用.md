# Druid的使用

### 安装

```
步骤一：下载
	官网https://imply.io/下载相应版本，本机使用imply-2.7.10.tar.gz
步骤二：
	使用sftp工具上传到VMWare主机hadoop02（192.168.8.105）的/opt目录并解压，重命名解压后的目录名称/opt/imply-2.7.10
步骤二：配置
	①配置zk集群地址
		①修改配置文件/opt/imply-2.7.10/conf/druid/_common/common.runtime.properties：
			druid.zk.service.host=hadoop02:2181,hadoop03:2181,hadoop04:2181
		②不使用内置zk
			修改/opt/imply-2.7.10/conf/supervise/quickstart.conf：
				:verify bin/verify-java
                #:verify bin/verify-default-ports
                #:verify bin/verify-version-check
                :kill-timeout 10
                #!p10 zk bin/run-zk conf-quickstart
步骤四：启动
	①启动zk集群
		zkServer-batch-start
	③启动imply
		bin/supervise -c conf/supervise/quickstart.conf
步骤五：验证
	访问hadoop02:9095
```

### 对比Presto/Spark SQL/Kylin/Elasticsearch

![计算框架对比图](D:\Git\MyRepository\Alex.blog.github.io\Images\计算框架对比图.png)

