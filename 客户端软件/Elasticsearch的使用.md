# Elasticsearch的使用

### 简介

```
Elasticsearch是一个基于Lucene的搜索服务器。它提供了一个分布式多用户能力的全文搜索引擎，基于RESTful web接口。
```

### 架构介绍

##### 节点类型

```
1. 候选主节点（Master-eligible node）
  一个节点启动后，就会使用Zen Discovery机制去寻找集群中的其他节点，并与之建立连接。集群中会从候选主节点中选举出一个主节点，主节点负责创建索引、删除索引、分配分片、追踪集群中的节点状态等工作。Elasticsearch中的主节点的工作量相对较轻，用户的请求可以发往任何一个节点，由该节点负责分发和返回结果，而不需要经过主节点转发。
  正常情况下，集群中的所有节点，应该对主节点的选择是一致的，即一个集群中只有一个选举出来的主节点。然而，在某些情况下，比如网络通信出现问题、主节点因为负载过大停止响应等等，就会导致重新选举主节点，此时可能会出现集群中有多个主节点的现象，即节点对集群状态的认知不一致，称之为脑裂现象。为了尽量避免此种情况的出现，可以通过discovery.zen.minimum_master_nodes来设置最少可工作的候选主节点个数，建议设置为(候选主节点数 / 2) + 1, 比如，当有三个候选主节点时，该配置项的值为(3/2)+1=2，也就是保证集群中有半数以上的候选主节点。
  候选主节点的设置方法是设置node.mater为true，默认情况下，node.mater和node.data的值都为true，即该节点既可以做候选主节点也可以做数据节点。由于数据节点承载了数据的操作，负载通常都很高，所以随着集群的扩大，建议将二者分离，设置专用的候选主节点。当我们设置node.data为false，就将节点设置为专用的候选主节点了。
    node.master = true
    node.data = false
    
2. 数据节点（Data node）
  数据节点负责数据的存储和相关具体操作，比如CRUD、搜索、聚合。所以，数据节点对机器配置要求比较高，首先需要有足够的磁盘空间来存储数据，其次数据操作对系统CPU、Memory和IO的性能消耗都很大。通常随着集群的扩大，需要增加更多的数据节点来提高可用性。
  前面提到默认情况下节点既可以做候选主节点也可以做数据节点，但是数据节点的负载较重，所以需要考虑将二者分离开，设置专用的数据节点，避免因数据节点负载重导致主节点不响应。
    node.master = false
    node.data = true

3. 客户端节点（Client node）
  按照官方的介绍，客户端节点就是既不做候选主节点也不做数据节点的节点，只负责请求的分发、汇总等等，也就是下面要说到的协调节点的角色。这样的工作，其实任何一个节点都可以完成，单独增加这样的节点更多是为了负载均衡。
    node.master = false
    node.data = false

4. 协调节点（Coordinating node）
  协调节点，是一种角色，而不是真实的Elasticsearch的节点，你没有办法通过配置项来配置哪个节点为协调节点。集群中的任何节点，都可以充当协调节点的角色。当一个节点A收到用户的查询请求后，会把查询子句分发到其它的节点，然后合并各个节点返回的查询结果，最后返回一个完整的数据集给用户。在这个过程中，节点A扮演的就是协调节点的角色。毫无疑问，协调节点会对CPU、Memory要求比较高。
```

### 索引原理

![1594341911406](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1594341911406.png)

具体过程：
①当一个文档被索引，它被加入到内存缓存，同时加到事务日志。这时新数据还不能被检索和查询。
②默认每1秒（如果对数据不敏感，可以设置成30秒）或者当内存缓存区中的数据快满了，会触发refresh，将缓存中的数据写入文件缓冲区
③当事务日志大小超过512M（或者达到30分钟，不过ES5.0版本后去除掉了这个条件），也会触发一次 refresh，将缓存数据刷新到文件缓冲区
④数据刷新到文件缓冲区后就可以对外提供索引服务
⑤最后将文件缓冲区数据持久化到磁盘

### 安装

```
在主机hadoop02上安装：
 步骤一：
	从官网 https://www.elastic.co/cn/downloads/elasticsearch 下载相应版本的gz包，本机使用的elasticsearch-5.6.2.tar.gz
步骤二：
	使用sftp工具上传到VMWare主机hadoop02（192.168.8.105）的/opt目录并解压，重命名解压后的目录名称elasticsearch-5.6.2
步骤三：配置信息
	①进入/opt/elasticsearch-5.6.2/bin,编辑elasticsearch:
		增加JAVA_HOME=/opt/jdk1.8.0_251
	②编辑/opt/elasticsearch-5.6.2/config/elasticsearch.yml
        #配置es的集群名称，用于区分不同的集群。
        cluster.name: my-es
        #节点名称
        node.name: node-1
        #设置索引数据的存储路径
        path.data: /opt/elasticsearch-5.6.2/data
        #设置日志的存储路径
        path.logs: /opt/elasticsearch-5.6.2/logs
        #设置当前的ip地址,通过指定相同网段的其他节点会加入该集群中
        network.host: 0.0.0.0
        #设置web服务的http端口
        http.port: 9200
        #设置集群中master节点的初始列表，可以通过这些节点来自动发现新加入集群的节点
        discovery.zen.ping.unicast.hosts: ["hadoop02","hadoop03","hadoop04"]
        #防止脑裂，设置集群中候选主节点的个数【(集群节点个数/2)+1】，有过半数的候选节点就可以保证选举出唯一的主节点
        discovery.zen.minimum_master_nodes: 2
    ③修改最大可创建文件数(启动es最低要求65536)，编辑/etc/security/limits.conf
    	末尾处增加：
    		*       soft    nofile  65536
            *       hard    nofile  65536
            *       soft    nproc   4096
            *       hard    nproc   4096
    ④修改最大虚拟内存（启动es最低要求262144），编辑/etc/sysctl.conf
    	①修改：vm.max_map_count=262144
    	②立即生效，sysctl -p
    ⑤修改jvm参数
        修改:
            -Xms256m
            -Xmx256m
        默认大小是2g，生产中可增加-Xms和-Xmx的值
步骤四：分发到hadoop03、hadoop04
	xsync /opt/elasticsearch-5.6.2
	xsync /etc/sysctl.conf
	xsync /etc/security/limits.conf
步骤五：配置hadoop03、haodoop04
	①hadoop03
		①使最大虚拟内存立刻生效，sysctl -p
		②修改hadoop03,hadoop/opt/elasticsearch-5.6.2/config/elasticsearch.yml中:node.name: node-2
	②hadoop04
		①使最大虚拟内存立刻生效，sysctl -p
		②修改hadoop/opt/elasticsearch-5.6.2/config/elasticsearch.yml中:node.name: node-3
步骤六：编辑群起脚本es-batch.sh
	#!/bin/bash
    #群起/群关elasticsearch集群

    case $1 in
    "start"){
    for i in hadoop02 hadoop03 hadoop04
    do
    echo "######启动$i es服务######"
    ssh $i "nohup /opt/elasticsearch-5.6.2/bin/elasticsearch > /dev/null 2>&1 &"
    done
    };;
    "stop"){
    for i in hadoop02 hadoop03 hadoop04
    do
    echo "######停止$i es服务######"
    ssh $i "nohup ps -ef | grep Elasticsearch | grep -v 'grep'|awk '{print $2}'| xargs kill > /dev/null 2>&1 &"
    done
    }
    esac
步骤七：启动
	es不能在root用户下启动，需要将以下文件/目录的所有者切换为alex
	①指定新的用户组和用户
		chown -R alex:alex /opt/elasticsearch-5.6.2
		chown alex:alex /usr/local/bin/es-batch.sh
	②启动
		es-batch.sh start
步骤七：验证
	①分别访问：hadoop02:9200、hadoop03:9200、hadoop04:9200，查看是否返回节点配置信息
	②jps查看，出现Elasticsearch进程
注意：es的服务端口是9300（在idea使用es客户端连接的就是9300）
	 es对外提供的web访问端口是9200
```

### 数据库比较

|                | redis   | mysql   | elasticsearch | hbase                | hive    |
| -------------- | ------- | ------- | ------------- | -------------------- | ------- |
| 容量           | 低      | 中      | 大            | 海量                 | 海量    |
| 查询速度       | 极快    | 中等    | 较快          | 中等                 | 慢      |
| 写入速度       | 极快    | 中等    | 较快          | 较快                 | 慢      |
| 特点           | 基于k-v | 支持sql | 全文检索      | 配合phoenix，支持sql | 支持sql |
| 一致性（事务） | 弱      | 强      | 弱            | 弱                   | 弱      |

### 常见术语

```
Cluster（集群）
Node（节点）
Index（索引，类似mysql数据库）
Type（类型，类似mysql表）
Document（文档，类似mysql行）
Field（字段，类似mysql列）
mapping（映射，类似mysql中的约束）
```

| 类型                           | 说明                                         |
| ------------------------------ | -------------------------------------------- |
| Cluster                        | 集群                                         |
| Node                           | 节点                                         |
| Index                          | 索引，类似mysql数据库                        |
| Type                           | 类型，类似mysql表                            |
| Document【由数据和元数据构成】 | 文档，类似mysql行                            |
| _index（Document元数据之一）   | 索引                                         |
| _type（Document元数据之一）    | 类型                                         |
| _id（Document元数据之一）      | 行的唯一标识（默认由es自动生成，也可以指定） |
| Field                          | 字段，类似mysql列                            |
| mapping                        | 映射，类似mysql中的约束                      |

### 可视化工具Kibana

简介

```
Kibana 是一款开源的数据分析和可视化平台，为es提供检索界面
```

安装

```
步骤一：下载
	进入/opt目录后，使用wget命令下载(跟es版本一致)：
	wget https://artifacts.elastic.co/downloads/kibana/kibana-5.6.2-linux-x86_64.tar.gz
步骤二：
	解压到当前目录/opt，重命名为kibana-5.6.2
步骤三：配置
	修改/opt/kibana-5.6.2/config/kibana.yml,编辑：
		#允许远程访问
		server.host: "0.0.0.0"
		#服务名，随意
		server.name: "my-kibana"
		#es web服务地址
		elasticsearch.url: "http://hadoop02:9200"
		#web服务端口（默认）
		server.port: 5601
步骤四：编辑启动脚本（与es服务一同启动）
```

```
#!/bin/bash
#群起/群关elasticsearch集群,以及kibana web服务端
case $1 in
"start"){
        for i in hadoop02 hadoop03 hadoop04
        do
                echo "######启动$i es服务######"
                ssh $i "nohup /opt/elasticsearch-5.6.2/bin/elasticsearch > /dev/null 2>&1 &"
        done
        echo "######启动 kibana 服务######"
        ssh hadoop02 "nohup /opt/kibana-5.6.2/bin/kibana > /dev/null 2>&1 &"
};;
"stop"){
        for i in hadoop02 hadoop03 hadoop04
        do
                echo "######停止$i es服务######"
                ssh $i "nohup ps -ef | grep 'Elasticsearch' | grep -v 'grep'|awk '{print $2}'| xargs kill > /dev/null 2>&1 &"
        done
        echo "######停止 kibana 服务######"
        ssh hadoop02 "nohup ps -ef | grep 'kibana' | grep -v 'grep' | awk '{print $2}'| xargs kill > /dev/null 2>&1 &"
}
esac
```

```
步骤五：启动
	切换到alex用户后，使用命令：es-batch.sh start
步骤六：验证
	访问http://hadoop02:5601
```

### Dev Tools常见命令

```
GET操作
	GET 索引名/_search，查看指定索引的所有document
	GET 索引名/_search{
		"query":{
			"match":{
				"filed名":"条件值"
			}
		}
	}，查看指定索引下的指定filed的一条或多条document
	
```

更多原理了解 https://www.jianshu.com/p/cc06f9adbe82 

生产调优了解 https://www.cnblogs.com/sanduzxcvbnm/p/12624662.html 