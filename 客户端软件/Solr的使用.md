# Solr的使用

简介

```
Solr是一个高性能，采用Java开发，基于Lucene的全文搜索服务器。
```

安装

```
步骤一：
	从官网https://lucene.apache.org/solr/downloads.html下载相应版本的tgz包，本机使用的solr-5.2.1.tgz
步骤二：
	使用sftp工具上传到VMWare主机hadoop02（192.168.8.105）的/opt目录并解压，重命名解压后的目录名称solr-5.2.1
步骤三：配置信息
	①在主机hadoop02，进入/opt/solr-5.2.1/bin,修改solr.in.sh
        #zk集群地址
        ZK_HOST="hadoop02:2181,hadoop03:2181,hadoop04:2181" 		
        #solr主机名
        SOLR_HOST="hadoop02" 
        #solr服务端口号 ，默认8983
        SOLR_PORT=8983
     ②分发到hadoop03,hadoop04
		xsync /opt/solr-5.2.1
	③修改hadoop03,/opt/solr-5.2.1/bin/solr.in.sh：
		SOLR_HOST="hadoop03"
		修改hadoop04,/opt/solr-5.2.1/bin/solr.in.sh：
		SOLR_HOST="hadoop04"
步骤四：启动服务
	①/opt/solr-5.2.1/bin start
	②编辑群起脚本solr-batch.sh：
#!/bin/bash
#solr集群群起/群关脚本

case $1 in
"start"){
        for i in hadoop02 hadoop03 hadoop04
        do
                echo -e "######启动$i solor web服务端口########"
                ssh $i "source /etc/profile;nohup /opt/solr-5.2.1/bin/solr start > /dev/null &"
        done
};;
"stop"){
        for i in hadoop02 hadoop03 hadoop04
        do
                echo -e "######停止$i solor web服务端口########"
                ssh $i "source /etc/profile;nohup /opt/solr-5.2.1/bin/solr stop > /dev/null &"
        done
}
esac
步骤五：验证
	①jps出现jar进行
	②访问http://hadoop02:8983/solr
```

