# HDFS的Shell操作

#### 准备工作

```
①HDFS集群包括VMWare上的主机hadoop02（192.168.8.105）、hadoop03（192.168.8.106）、hadoop03（192.168.8.107），三个节点集群的部署规划
	##################################################
			hadoop02	hadoop03 		hadoop04
	##################################################
	HDFS	NameNode	DataNode		SecondaryNameNode
			DataNode					DataNode
	##################################################
	YARN	NodeManager	NodeManager		NodeManager
						ResourceManager
	##################################################
②在hadoop02下启动集群的NameNode和DataNode，使用命令/bin/start-dfs.sh
在hadoop03下启动集群的ResourceManager和NodeManager，使用命令/bin/start-yarn.sh
```

#### 基本语法

```
bin/hadoop fs 具体命令	或者	bin/hdfs dfs 具体命令
```

#### 命令（重点）

```
与linux类似的命令：
	[-cat [-ignoreCrc] <src> ...]，查看HDFS中/usr/alex目录中的test.txt内容
		hadoop fs -cat /usr/alex/test.txt
		
    [-chown [-R] [OWNER][:[GROUP]] PATH...]，将/usr/alex/test.txt文件的拥有者改为supergroup:alex
    	①HDFS本身的组和用户都是同步Linux系统的，默认supergroup组，首先在linux系统下创建alex用户
    		useradd alex
    	②在root用户下使用命令：hadoop fs -chown alex:supergroup /usr/alex/test.txt
		
	[-chmod [-R] <MODE[,MODE]... | OCTALMODE> PATH...]，将/usr/alex/test.txt权限更改rwx-rw-rw
		hadoop fs -chmod 766 /usr/alex/test.txt
	
	[-chgrp [-R] GROUP PATH...]，更改/usr/alex/test.txt的组为alex
		hadoop fs -chgrp alex /usr/alex/test.txt

    [-cp [-f] [-p | -p[topax]] <src> ... <dst>]，其中src和dst都是HDFS系统目录，将HDFS 下/usr/alex/test.txt 拷贝到上一级目录
    	hadoop fs -cp /usr/alex/test.txt /usr/
    	
    [-mv <src> ... <dst>]，其中src和dst都是HDFS系统目录，将/usr/alex/test.txt 移动到上一级目录
    	hadoop fs -mv /usr/alex/test.txt /usr/
    	
    [-rm [-f] [-r|-R] [-skipTrash] <src> ...]，删除指定的文件或者目录，删除/usr/alex/logs目录
    	hadoop fs -rm -R /usr/alex/logs

    [-put [-f] [-p] [-l] <localsrc> ... <dst>]，将linux系统的文件/目录上传到HDFS系统,将linux本地文件/home/crontab/job.sh上传到HDFS /usr/alex下
    	hadoop fs -put /home/crontab/job.sh /usr/alex
    
    [-get [-p] [-ignoreCrc] [-crc] <src> ... <localdst>]，从HDFS上下载/usr/alex/test.txt到本地/home
    	hadoop fs -get /usr/alex/test.txt /homes
   
    [-du [-s] [-h] <path> ...]，查看/usr/alex下的所有文件的大小
    	hadoop fs -h /usr/alex
    	
	[-ls [-d] [-h] [-R] [<path> ...]]，查看/usr/alex下的目录所有文件信息
		hadoop fs -ls -h /usr/alex
		
	[-find <path> ... <expression> ...],查找/usr/alex下名为test.txt的文件
		hadoop fs -find /usr/alex/test.txt
		
特有的命令：(注意hadoop-2.7.2还未实现moveFromLocal和moveToLocal)
	[-getmerge [-nl] <src> <localdst>],从HDFS下载合并文件到本地，将/usr/alex/logs下的所有文件合并下载到本地/home/logs/log_merge.log
		hadoop fs -getmerge /usr/alex/logs/* /home/logs/log_merger.log
	
	[-moveFromLocal <localsrc> ... <dst>],从linux本地将文件移动到HDFS目录，将/home/logs/log_merge.log移动到/usr/alex/logs
		hadoop fs -moveFromLocal /home/logs/log_merge.log /usr/alex/logs
	
    [-moveToLocal <src> <localdst>]，从HDFS下将文件移动到linux本地目录，将/usr/alex/logs/log_merge.log 移动到/home/logs/log_merge.log
    	hadoop fs -moveToLocal /usr/alex/logs/log_merge.log /home/logs/log_merge.log
    
    [-copyFromLocal [-f] [-p] [-l] <localsrc> ... <dst>]从linux本地将文件复制到HDFS目录，将/home/logs/log_merge.log复制到/usr/alex/logs
		hadoop fs -copyFromLocal /home/logs/log_merge.log /usr/alex/logs
    
    [-copyToLocal [-p] [-ignoreCrc] [-crc] <src> ... <localdst>]，从HDFS下拷贝文件到本地目录将/usr/alex/logs/log_merge.log 复制到/home/logs/log_merge.log
    	hadoop fs -copyToLocal /usr/alex/logs/log_merge.log /home/logs/log_merge.log
    	
    [-setrep [-R] [-w] <rep> <path> ...],设置DataNode副本数，设置/usr/alex/test.txt的副本数是10
    	hadoop fs -setrep 10 /usr/alex/test.txt
```

### 集群间拷贝

```
distcp是用于大规模集群内部或者集群之间的高性能拷贝工具，和在linux上执行cp，scp实现效果是一致的，不同的是，cp是将本机的文件和目录拷贝到本机的其它地方，scp则可以将A机器的文件或者目录拷贝到B机器，而Distcp则可以实现的是A（hdfs）集群的数据拷贝到B（hdfs）集群，而分布式使得数据拷贝时，可以实现A级群的DN节点同时向B集群的DN节点发送数据，突破了单机拷贝的网卡速率限制，拷贝效率更高。

基本语法：
	hadoop distcp 源地址 目的地址
	比如将集群1上的文件/usr/alex/test.txt（hdfs://${fs.defaultFS}/usr/alex/test.txt）拷贝到集群2上（hdfs://${fs.defaultFS}/usr/alex/test.txt）
	hadoop distcp hdfs://${fs.defaultFS}/usr/alex/test.txt hdfs://${fs.defaultFS}/usr/alex/test.txt
	注意${fs.defaultFS}根据所在集群的真实地址填写
```

### 自定义Shell脚本

xsync脚本（封装rsync）实现数据同步下发：

```
#!/bin/bash
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
        echo $pdir/$fname $user@hadoop0$host:$pdir
        rsync -rvl $pdir/$fname $user@hadoop0$host:$pdir
done
#Note:这里的@hadoop0$host是拼接主机名，需要做相应修改。另外，for循环中的host的边界值由自己的主机编号决定。
```

批量开启zkServer服务

```
#!/bin/bash
# 批量开启zookeeper集群
for i in  2 3 4
do
echo -e ""
echo -e "starting hadoop0$i zkServer"
ssh hadoop0$i "source /etc/profile;$ZOOKEEPER_HOME/bin/zkServer.sh start"
done
```

批量查看集群所有节点的java进程

```
#!/bin/bash
# 批量查看hadoop集群中的java进程
for i in 2 3 4
do
echo -e "##########hadoop0$i java进程#################"
ssh hadoop0$i "source /etc/profile;$JAVA_HOME/bin/jps"
done
```

### 案例

##### 案例一：群起flume采集日志脚本

------

#脚本/usr/local/bin/file-flume-kafka.sh，来自电商数仓项目

#!/bin/bash
#群起/群关flume日志采集功能

case $1 in
"start"){
        for i in hadoop02 hadoop03
        do
                echo "#######开启 $i flume采集功能#######"
                ssh $i "nohup /opt/flume-1.7.0/bin/flume-ng agent --name myagent --conf-file /opt/flume-1.7.0/cases/case5/file-flume-kafka.conf -Dflume.root.logger=DEBUG,LOGFILE >/dev/null **2>&1** &"
        done
};;
"stop"){
        for i in hadoop02 hadoop03
        do
                pid=$(ssh $i "ps -ef|grep 'file-flume-kafka'|grep -v 'grep'|awk 'NR==1{print \$2}'")
                echo "#######关闭 $i flume采集功能#######"
                if [ "$pid" ]
                then
                        ssh $i kill $pid
                fi
        done
};;
esac

------

##### 案例二：flume 消费kafka日志脚本

------

#脚本/usr/local/bin/kafka-flume-hdfs.sh，来自电商数仓项目

#!/bin/bash
#群起/群关flume消费kafka日志功能
case $1 in
"start"){
        for i in hadoop04
        do
                echo "#######开启 $i flume消费kafka日志功能#######"
                ssh $i "nohup /opt/flume-1.7.0/bin/flume-ng agent --name myagent1 --conf-file /opt/flume-1.7.0/cases/case5/kafka-flume-hdfs.conf -Dflume.root.logger=DEBUG,LOGFILE > /dev/null 2>&1 &"
        done
};;
"stop"){
        for i in hadoop04
        do
                pid=$(ssh $i "ps -ef|grep 'kafka-flume-hdfs'|grep -v 'grep'|awk 'NR==1{print \$2}'")
                echo "#######关闭 $i flume消费kafka日志功能#######"
                if [ "$pid" ]
                then
                        ssh $i kill $pid
                fi
        done
};;
esac

------

```
案例知识点：
	【nohup命令】：nohup Command [ Arg … ] &
	【作用】：在关闭终端或者退出账户时程序依然保持后台运行（除非被杀掉），默认情况下，会在当前目录下自动生成nohup.out文件记录程序运行过程中的日志
	
	【ps grep配合awk命令】：ps -ef|grep pid/进程名 |grep -v 'grep'| awk '正则1&&正则2&&NR==1{print $第几列}'
		NR==1，从正则匹配后的记录中取第一行记录
	【作用】：常用于获取进程pid
	
	【/dev/null】/dev/null代表linux的空设备文件，所有往这个文件里面写入的内容都会丢失，俗称“黑洞”
        标准输入0：从键盘获得输入 /proc/self/fd/0 
        （默认）标准输出1：输出到屏幕（即控制台） /proc/self/fd/1 
        错误输出2：输出到屏幕（即控制台） /proc/self/fd/2
        
     分为以下三种使用情况：   
     2>/dev/null
     	表示：把错误输出到“黑洞”
     2>&1
     	表示：把错误输出2重定向到标准输出1，而由于默认情况下，标准输出1就是重定向到”黑洞“，因此，最终错误输出2和标准输出1都进了“黑洞”，不会打印到控制台
     2>&1 >/dev/null
		表示：把错误输出2重定向到标准输出1，也就是屏幕，标准输出进了“黑洞”，最终标准输出进了黑洞，错误输出打印到屏幕
```

