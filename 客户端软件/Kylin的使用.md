# Kylin的使用

### 安装

```
在主机hadoop02安装：
步骤一：
	从官网http://kylin.apache.org/download/下载相应版本的gz包，本机使用的apache-kylin-2.3.2-bin.tar.gz
步骤二：
	使用sftp工具上传到VMWare主机hadoop02（192.168.8.105）的/opt目录并解压，重命名解压后的目录名称kylin-2.3.2
步骤三：配置信息
	①进入/opt/kylin-2.3.2,将当前目录配置到系统环境变量中：
        ①编辑/etc/profile文件，增加
                export KYLIN_HOME=/opt/kylin-2.3.2
                export PATH=$PATH:$KYLIN_HOME/bin
		②使配置立刻生效,source /etc/profile
步骤四：启动
	①需要依次启动
        start-dfs.sh //启动hdfs集群
        start-yarn.sh	//启动yarn集群
        mr-jobhistory-daemon.sh start historyserver  //启动mapreduce历史服务器
        zkServer-batch-start	//启动zookeeper集群
        start-hbase.sh	//启动hbase集群
    ②启动kylin，使用命令：kylin.sh start
    ③访问http://hadoop02:7070/kylin
```

