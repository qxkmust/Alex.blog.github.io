# Zabbix的使用

### 简介

```
zabbix是一个基于WEB界面的提供分布式系统监视以及网络监视功能的企业级的开源解决方案，提供灵活的通知机制，帮助管理人员快速定位问题。类似的集群监控还有ganglia
```

### 架构

![1594790331133](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1594790331133.png)

### 安装

```
步骤一：下载
	官网https://www.zabbix.com/download_sources下载相应版本，本机使用zabbix-4.2.8.tar.gz
步骤二：
	使用sftp工具上传到VMWare主机hadoop02、hadoop03、hadoop04的/opt目录并解压，重命名解压后的目录名称/opt/zabbix-4.2.8
步骤三：准备工作
	配置主机【hadoop02】：
	①关闭防火墙
		systemctl stop firewalld
		systemctl disable firewalld
		chkconfig firewalld off
	②关闭SELinux
		修改/etc/selinux/config
			SELINUX=disabled
	③安装mysql的完整组件(client和server已经安装)
		rpm -ivh MySQL-shared-compat-5.6.49-1.el7.x86_64.rpm
		rpm -ivh MySQL-shared-5.6.49-1.el7.x86_64.rpm
		rpm -ivh MySQL-devel-5.6.49-1.el7.x86_64.rpm
		rpm -ivh MySQL-embedded-5.6.49-1.el7.x86_64.rpm
		注意：这里用mysql5.6版本，5.5版本无法支持zabbix
	④安装依赖
		yum install -y libcurl libcurl-devel libxml2 libxml2-devel net-snmp-devel libevent-devel pcre-devel gcc-c++
	⑤创建用户和组(zabbix不能运行在root用户下，需要创建zabbix用户)
		groupadd --system zabbix
		useradd --system -g zabbix -d /usr/lib/zabbix -s /sbin/nologin -c "Zabbix Monitoring System" zabbix
	⑥编译安装
		编译：./configure --enable-server --enable-agent --with-mysql --enable-ipv6 --with-net-snmp --with-libcurl --with-libxml2
		安装：make install
		
	以下配置主机【hadoop03和hadoop04】：
	①创建用户和组(zabbix不能运行在root用户下，需要创建zabbix用户)
		groupadd --system zabbix
		useradd --system -g zabbix -d /usr/lib/zabbix -s /sbin/nologin -c 
	②准备编译环境
		yum -y install gcc-c++ pcre-devel
	③编译及安装
		编译：./configure --enable-agent
		安装：make install
步骤四：主机【hadoop02】配置zabbix-server
	①编辑/usr/local/etc/zabbix_server.conf
		DBHost=hadoop02
		DBName=zabbix
		DBUser=alex
		DBPassword=alex123
	②编辑开机启动脚本/etc/init.d/zabbix-server
```

```
#!/bin/sh
#
# chkconfig: - 85 15
# description: Zabbix server daemon
# config: /usr/local/etc/zabbix_server.conf
#
### BEGIN INIT INFO
# Provides: zabbix
# Required-Start: $local_fs $network
# Required-Stop: $local_fs $network
# Default-Start:
# Default-Stop: 0 1 2 3 4 5 6
# Short-Description: Start and stop Zabbix server
# Description: Zabbix server
### END INIT INFO
# Source function library.
. /etc/rc.d/init.d/functions
if [ -x /usr/local/sbin/zabbix_server ]; then
exec=/usr/local/sbin/zabbix_server
else
exit 5
fi
prog=zabbix_server
conf=/usr/local/etc/zabbix_server.conf
pidfile=/tmp/zabbix_server.pid
timeout=10
if [ -f /etc/sysconfig/zabbix-server ]; then
. /etc/sysconfig/zabbix-server
"/etc/rc.d/init.d/zabbix-server" 82L, 1463C
start|stop|restart)
$1
;;
force-reload)
restart
;;
status)
status -p $pidfile $prog
;;
try-restart|condrestart)
if status $prog >/dev/null ; then
restart
fi
;;
reload)
action $"Service ${0##*/} does not support the reload action:
" /bin/false
exit 3
;;
*)
echo
$"Usage:
$0
{start|stop|status|restart|try-restart|force-reload}"
exit 2
;;
esac
```

```
	②设置开机启动
		chkconfig --add zabbix-server
		chkconfig zabbix-server on
步骤五：主机【hadoop02-hadoop04】配置zabbix-agent
	①编辑/usr/local/etc/zabbix_agentd.conf
		Server=hadoop02
		#ServerActive=127.0.0.1
		#Hostname=Zabbix server
	②编辑开机启动脚本/etc/init.d/zabbix-agent
```

```
#!/bin/sh
#
# chkconfig: - 86 14
# description: Zabbix agent daemon
# processname: zabbix_agentd
# config: /usr/local/etc/zabbix_agentd.conf
#
### BEGIN INIT INFO
# Provides: zabbix-agent
# Required-Start: $local_fs $network
# Required-Stop: $local_fs $network
# Should-Start: zabbix zabbix-proxy
# Should-Stop: zabbix zabbix-proxy
# Default-Start:
# Default-Stop: 0 1 2 3 4 5 6
# Short-Description: Start and stop Zabbix agent
# Description: Zabbix agent
### END INIT INFO
# Source function library.
. /etc/rc.d/init.d/functions
if [ -x /usr/local/sbin/zabbix_agentd ]; then
exec=/usr/local/sbin/zabbix_agentd
else
exit 5
fi
prog=zabbix_agentd
conf=/usr/local/etc/zabbix_agentd.conf
pidfile=/tmp/zabbix_agentd.pid
"/etc/rc.d/init.d/zabbix-agent" 85L, 1563C
start|stop|restart)
$1
;;
force-reload)
restart
;;
status)
status -p $pidfile $prog
;;
try-restart|condrestart)
if status $prog >/dev/null ; then
restart
fi
;;
reload)
action $"Service ${0##*/} does not support the reload action:
" /bin/false
exit 3
;;
*)
echo
$"Usage:
$0
{start|stop|status|restart|try-restart|force-reload}"
exit 2
;;
esac
```

```
	③分发到hadoop03和hadoop04
	④设置开机启动
		chkconfig --add zabbix-agent，加到开机启动列表
		chkconfig zabbix-agent on，开机启动
步骤六：主机【hadoop02】配置zabbix-web
	①安装httpd，yum install -y httpd
	②配置httpd，编辑/etc/httpd/conf/httpd.conf,在<Directory "/var/www/html">节点下增加：
		<IfModule mod_php5.c>
            php_value max_execution_time 300
            php_value memory_limit 128M
            php_value post_max_size 16M
            php_value upload_max_filesize 2M
            php_value max_input_time 300
            php_value max_input_vars 10000
            php_value always_populate_raw_post_data -1
            php_value date.timezone Asia/Shanghai
    	</IfModule>
    ③安装php及依赖
    	yum install -y php php-bcmath php-mbstring php-xmlwriter php-xmlreader php-mcrypt php-cli php-gd php-curl php-mysql php-ldap php-zip php-fileinfo
    ③设置开机启动
    	chkconfig httpd on
步骤七：启动
	主机hadoop02：
        ①启动zabbix-server
            service zabbix-server start
        ①启动zabbix-agent
            service zabbix-agent start
        ①启动httpd
            service httpd start
    主机hadoop03和hadoop04：
    	启动zabbix-agent
            service zabbix-agent start
步骤八：验证
	①访问http://hadoop02/zabbix，进入初始化配置页面，配置主机名，数据库等等信息
	②之后使用用户名：Admin，密码：zabbix登录
```

### 使用案例

##### 案例一：监控hadoop集群

```
需求：
	①监控hadoop02中的namenode
	②监控hadoop03中的datanode
	③监控hadoop04中的secondarynamenode
解决：
步骤一：创建主机Host
	在zabbix监控页 -> 配置 -> 主机 -> 创建主机，分别创建hadoop02（192.168.8.105）、hadoop03（192.168.8.106）、hadoop04（192.168.8.107）
步骤二：创建监控项Item
	①选中主机hadoop02 -> 监控项 -> 创建监控项 -> 键值：proc.num[,,all,namenode]
	①选中主机hadoop03 -> 监控项 -> 创建监控项 -> 键值：proc.num[,,all,datanode]
	①选中主机hadoop04 -> 监控项 -> 创建监控项 -> 键值：proc.num[,,all,secondarynamenode]
步骤三：创建触发器
	①选中主机hadoop02 -> 触发器 -> 创建触发器 -> 表达式：	{hadoop02:proc.num[,,all,namenode].last()}=0
	①选中主机hadoop03 -> 触发器 -> 创建触发器 -> 表达式：	{hadoop02:proc.num[,,all,datanode].last()}=0
	①选中主机hadoop04 -> 触发器 -> 创建触发器 -> 表达式：	{hadoop02:proc.num[,,all,secondarynamenode].last()}=0
步骤四：创建动作
	配置 -> 动作 -> 创建动作 -> 分别指定对应的触发器
步骤五：配置发送者邮箱
	①这里使用QQ邮箱，需要在QQ邮箱中开启POP3/SMTP服务，获取到第三方客户端登录密码：hsdwpnqpdxlsbccc，这个密码就是下边的认证密码
	②管理 -> 报警媒介类型 -> 内容如下
```

![1594966052456](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1594966052456.png)

```
步骤六：配置接收者邮箱
	用户 -> 报警媒介 -> 内容如下
```

![1594960691272](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1594960691272.png)

```
步骤七：
①在hadoop02先群起hadoop集群：start-dfs.sh,之后群关：stop-dfs.sh
②观察zabbix监控页是否出现集群告警，是否接收到告警邮件
```

#### 案例二：使用模板

```
生产中往往需要监控大量服务器中的几十种监控项，如果单独配置每个服务器的每个监控项将耗费大量的人力，使用模板可以解决这个问题
```

```
步骤一：创建主机Host
	在zabbix监控页 -> 配置 -> 主机 -> 创建主机，分别创建hadoop02（192.168.8.105）、hadoop03（192.168.8.106）、hadoop04（192.168.8.107）
步骤二：创建模板
	配置 -> 模板 -> 创建模板 -> 分别创建模板hadoop_namenode_monitor_template、hadoop_datanode_monitor_template、hadoop_2nn_monitor_template
步骤二：创建监控项Item
	①选中模板hadoop_namenode_monitor_template -> 监控项 -> 创建监控项 -> 键值：proc.num[,,all,namenode]
	①选中hadoop_datanode_monitor_template -> 监控项 -> 创建监控项 -> 键值：proc.num[,,all,datanode]
	①选中hadoop_2nn_monitor_template -> 监控项 -> 创建监控项 -> 键值：proc.num[,,all,secondarynamenode]
步骤三：创建触发器
	①选中模板hadoop_namenode_monitor_template -> 触发器 -> 创建触发器 -> 表达式：	{hadoop02:proc.num[,,all,namenode].last()}=0
	①选中hadoop_datanode_monitor_template -> 触发器 -> 创建触发器 -> 表达式：	{hadoop02:proc.num[,,all,datanode].last()}=0
	①选中hadoop_2nn_monitor_template -> 触发器 -> 创建触发器 -> 表达式：	{hadoop02:proc.num[,,all,secondarynamenode].last()}=0
步骤四：创建动作
	配置 -> 动作 -> 创建动作 -> 分别指定对应模板的触发器
其它步骤同案例一
```

