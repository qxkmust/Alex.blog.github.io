# Logstash的使用

### 简介

```
Logstash是一个开源数据收集引擎，具有实时管道功能。Logstash可以动态地将来自不同数据源的数据统一起来，并将数据标准化到你所选择的目的地。一般输出到Elasticsearch
```

### 安装

```
步骤一：下载
	官网https://www.elastic.co/cn/downloads/logstash下载相应版本，本机使用logstash-5.6.2.tar.gz
步骤二：
	使用sftp工具上传到VMWare主机hadoop02（192.168.8.105）的/opt目录并解压，重命名解压后的目录名称/opt/logstash-5.6.2
步骤二：配置
	①进入/opt/logstash-5.6.2,将当前目录配置到系统环境变量中：
        ①编辑/etc/profile文件，增加
                export LOGSTASH_HOME=/opt/logstash-5.6.2
                export PATH=$PATH:$LOGSTASH_HOME/bin
		②使配置立刻生效,source /etc/profile
步骤四：启动
	/bin/logstash
	【命令1】logstash -f 【配置文件路径】
	【命令2】logstash -e 【以命令行指定配置】，logstash -e 'input {stdin{}} output {stdout{}}'
```

### 配置文件

```
logstash包含三个过滤器，
    input过滤器：采集各种样式、大小和来源的数据
    filter过滤器：实时解析和转换数据
    output过滤器：选择要存储的方式
在配置文件中编辑过滤器规则，格式如下：
# 日志导入
input {
}
# 日志筛选匹配处理
filter {
}
# 日志匹配输出
output {
}
```

```
input包含多种输入源：
	websocket：端口
	stdin：控制台输入
	jdbc：来自jdbc
	file：外部文件
	elasticsearch：读取es数据源
	kafka：来自kafka 主题
	redis:来自redis
```

```
filter包含丰富的过滤器，可以进行数据解析、删除字段、类型转换
过滤器：
	date：日期解析
	grok：正则匹配解析
	dissect：分割符解析
	mutate：对字段进行重命名、删除、替换等
	json：按照json格式解析字段内容到指定字段
	geoip：增加地理位置数据
	ruby：利用ruby代码动态修改event
```

```
output包含多种输出源：
	stdout：控制台
	file：输入到文件
	kafka：写入kafka主题
	elasticsearch：es索引
	redis：写入缓存
```

### 案例

#### 案例一：监控文件变化

```
①在/opt/logstash-5.6.2/cases/case1新建配置file-to-console.conf，编辑：
input{
    file{
        path=>["/opt/logstash-5.6.2/cases/case1/*.txt"]  //可以指定多目录，多文件
        start_position=>"beginning" //默认从头开始读
        exclude=>["*.cnf"] //过滤掉指定文件类型
        stat_interval=>1 //每隔1秒监控文件变化
    }
}
output{
        stdout{ //打印到控制台
        	codec => "rubydebug" //指定编码格式
        }
}
```

#### 案例二：采集es

```
①在/opt/logstash-5.6.2/cases/case2新建配置es-to-console.conf，编辑：
input{
    elasticsearch{
        hosts => "192.168.8.105" //主机地址
        index => "my_index" //索引
        query => '{ "query": { "match_all": {}}}' //es的查询语句
    }
}
output{
    stdout{
    	codec => "rubydebug" 
    }
}
```

#### 案例三：数据清洗

```
①在/opt/logstash-5.6.2/cases/case3新建配置date-filter.conf，编辑：
input{
    file{
        path => "/opt/logstash-5.6.2/cases/case3/date.json"
        codec => "json"
    }
}
filter{
    date{
    	//校验date属性值是否满足日期格式yyyy-MM-dd
    	//如果满足格式，默认将匹配到的日期赋值给timestamp属性
    	//如果不满足格式，会在tags属性显示“_dateparsefailure”错误信息
    	match => ["date","yyyy-MM-dd"] 
    	target => "@timestamp" //这句可以不写，默认赋值给timestamp属性
    }
}
output{
		//将满足日期格式的数据源写入到es
		//不满足的被过滤掉
        elasticsearch{
                hosts => "192.168.8.105"
                index => "logstash_to_es"
        }
        //同时打印到控制台
        stdout{
                codec => "rubydebug"
        }
}
```

### Logstash与Hive对比

|          | Logstash                 | Hive                                   |
| -------- | ------------------------ | -------------------------------------- |
| 组件     | input、filter、output    | source、channel、sink                  |
| 使用方式 | logstash -f 配置文件     | flume-ng agent -n 代理名称 -f 配置文件 |
| 事务     | 不支持                   | 支持                                   |
| 数据清洗 | filter过滤器             | 自定义拦截器                           |
| 场景     | kafka -> 采集、清洗 ->es | kafka -> 分类、打标签->hdfs            |

