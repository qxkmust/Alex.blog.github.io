# MYSQL常见问题

- 在linux系统下安装mysql

  ```
  方式一：二进制包安装-rpm安装
  1.查看当前linux系统的版本 uname -a
  2.查看是否已经安装mysql，rpm -qa|grep -i mysql,安装过，可以删除，rpm -e rpm软件包名
  3.登录官网https://dev.mysql.com/downloads/mysql/，下载相应linux系统版本的mysql的rpm安装包，
  一般centos（服务器操作系统）对应Red Hat Enterprise Linux，Ubuntu（图形化的桌面操作系统）对应Ubuntu Linux，Debian（另一款服务器操作系统）对应Debian Linux
  4.安装rpm文件包
  	①将下载的rpm文件（如果是mysql5版本只需要安装服务端和客户端，如果是8版本，需要安装其他更多依赖包）上传到linux的opt/目录下
  	②安装命令rpm -ivh rpm文件名
  		一般安装mysql服务端rpm就可以了，如果需要在服务器上连接mysql，还需要安装mysql 客户端rpm
  	③测试是否安装成功，rpm -qa|grep mysql或者ps -ef |grep mysql
  	注意：如果有包冲突，可先卸载冲突的包rpm -e 冲突的rpm包 --nodeps
  5.启动mysql服务 service mysql start
  6.默认mysql服务未开启远程连接，需要创建远程连接的用户：
  	①连接mysql服务
  		①mysql5版本，默认缺省密码为空，通过root用户可以直接连接mysql服务
  		②mysql8版本，默认缺省密码不为空，可以在/etc/my.cnf中[mysqld]下增加配置skip-grant-tables暂时跳过密码验证
  	②删除mysql库中user表中user为空的记录，一般只保留一个记录（user=root,host=localhost）
  	③默认的root用户设置密码以及增加权限
  		①mysql5版本，使用命令update user set password = password('任意密码') where user = 'root'
  		授权root用户可使用新的密码登录：
  			grant all privileges on *.* to '用户名'@'主机' identified by '密码' with grant option;
  		②mysql8版本，使用命令alter user '用户名'@'主机名' identified with mysql_native_password by '特殊密码'，这里的特殊密码必须满足长度>=8，且是英文和数字的组合，可以通过设置全局变量更改validate_password.length=1（配置后也必须满足长度>5才行）和validate_password.policy=0，建议写在配合文件中
  			①授权root用户可使用新的密码登录：
  				alter user '用户名'@'主机' identified with mysql_native_password by '密码';		
          		grant all privileges on *.* to '用户名'@'主机名'
  			②之后登出客户端，在my.cnf去除skip-grant-tables，重新连接mysql服务
  			③flush privileges，使上一次更改立刻生效，否则会发生错误： The MySQL server is running with the --skip-grant-tables option so it cannot execute this statement
  			
  	④用root用户登录，在mysql数据库的user表中增加一个用户并授权，使用命令:
  		①mysql5版本
              新增用户：create user 用户名 identified by '密码'
              授权：
              	①grant all privileges on *.* to '用户名'@'主机' identified by '密码' with grant option;
              	②flush privileges
          ②mysql8版本 
          	新增用户：create user 用户名 identified by '密码'
          	授权：
          		①alter user '用户名'@'主机' identified with mysql_native_password by '密码';		
          		②grant all privileges on *.* to '用户名'@'主机名'
          		③flush privileges;
  	⑤后查看user表中，创建了host为%的用户，支持远程连接
  	-------------------------------------------
  	host		user		password
  	-------------------------------------------
  	%			用户名			密码（非明文）
  	-------------------------------------------
  	
  方式二：源码包安装
  	这种方式需要先配置编译再安装，比较麻烦
  ```

- mysql主从复制如何配置

  ```
  一：概念和原理：
  	①mysql主从复制就是建立一个和主数据库完全一样的数据库环境（从数据库），从数据库可以实现主数据库的数据热备份（作用就是容灾备份），一个主数据库可以对应多个从数据库。
  	②原理：
  		①master将增删改操作记录到二进制日志（bin log）
  		②slave将master的binlog拷贝到它的中继日志（relay log）
  		③salve重做中继日志中的事件，将改变应用到自己的数据库，实现对master数据库的异步串行化复制
  二：配置mysql主从服务器
  	注意：①主从服务器可以位于不同的物理机，通过远程的方式互相访问
  		 ②主、从数据库的mysql版本必须一致，若主数据库是window版本，从数据库是linux版本，只要保证mysql的版本是同一个大版本就可以，比如都是5或都是8，以下针对mysql8版本的一主一从数据库.
  	环境：
  		①主数据库是window上mysql8.0.18版本，使用物理机，从数据库是虚拟机linux centos上mysql8.0.19版本，主机ip192.168.8.100，从机ip 192.168.8.103（与主机网络桥接），保证能互相通信（另外针对mysql5.5.52版本，将192.168.8.101部署为主机，192.168.8.102部署为从机）
          以下针对mysql8版本的一主一从数据库.
  	步骤一：配置主数据库
  		①在主机mysql的安装目录主配置文件mysql.ini中[mysqld]下增加：
  			①必填，主服务器ID server-id=1
  			②必填，启用二进制日志 log-bin=mysql根目录\data\binlog
  			③非必填，启用错误日志 log-err=mysql根目录\data\.err文件
  			④非必填，日志记录的数据类型 binlog_format=类型
  			类型有三种：
  				①row:将主机执行写操作后的数据记录到日志
  					优点：能保证数据一致
  					缺点：数据大量更新时，将大量的结果写到日志，会造成日志急剧增加
  				②（默认）statement：将写操作的那条sql记录到日志（不使用这种，相同的sql在主机和从机执行结果可能不一致）
  					优点：每次写操作只会记录一条sql，不会生成大量日志
  					缺点：可能造成主机/从机数据不一致
  				③（生产中推荐）mixed：根据执行的每一条具体的 SQL 语句来区分对待记录的日志形式，也就是在 statement 和 row 之间选择一种。
  			⑤非必填，指定哪个数据库的写操作需要记录到log-bin日志（默认mysql服务下所有的数据库） binlog-do-db=数据库名
  		②为从机访问主机创建一个单独的用户
  			①新增用户：create user 用户名 identified by '密码'
          	②为用户增加密码：alter user '用户名'@'主机' identified with mysql_native_password by '密码'，主机100为从机103分配的用户是alex，密码alex123;
          	③测试从机是否能使用该用户名和密码远程连接主机数据库
          		mysql -u 用户名 -h 主机ip -p
          	④将用户授权给从数据库使用
          		grant replication slave on *.* to 用户名@'主机'
          		注意：用户名不需要单引号，主机需要单引号
          	⑤使设置生效
          		flush privileges
          ③查看主机的二进制日志配置是否生效
          	使用命令：show master status;
          	以下两项有数据显示说明正常，若为empty，检查主从配置文件中的的log-bin：
          	___________________________________
          	File						Position 
          	记录主数据库操作的二进制日志文件	起始位置号
          	___________________________________
          
  	步骤二：配置从数据库
  		①在从机mysql服务的配置文件/etc/my.cnf中[mysqld]下增加：
  			①必填，从服务器ID server-id=2（只要与主服务器ID不同都行）
  			②必填，启用二进制日志 log-bin=mysql-bin
  		②设置从服务器链接到master主服务器：
  			change master to master_host='主机地址',master_user='用户名',master_password='密码',master_log_file='二进制日志文件名',master_log_pos=二进制日志中的起始位置数字
  			注意：master_log_file填步骤一③中的File的值，master_log_pos填Position的值
  		③启动从机服务，使用命令start slave
  			原理：同时启动I/O线程和SQL线程，I/O线程从主库读取bin log，并存储到relay log中继日志文件中，SQL线程读取中继日志，解析后，在从库执行一遍。
  		④检查从机服务是否正常
  			使用命令：show slave status\G;
              第一次使用start slave;show slave status\G;
              如果显示异常:	SLAVE_IO_STATE:CONNECTING TO MASTER HOST;
                          SLAVE_IO_RUNNING:NO;
                          SLAVE_SQL_RUNNING:NO
              可能原因：主从数据库/var/lib/mysql/auto.cnf中server-uuid相同
              解决：删除从数据库/var/lib/mysql/auto.cnf
              
       步骤三：验证主从复制功能
       		在主数据库进行增删改操作，同样的操作会同步到从数据库中
  ```

  

- 关闭大小写敏感开关

  ```
  mysql服务端默认开启表名大小写敏感开关，即lower_case_table_names=0
  通过客户端查看：
  	Navicat for Mysql或者mysql.exe工具连接服务端，使用语句show variables like 'lower%',显示如下
  ```

  | lower_case_file_system | OFF  | 只读参数，无法被修改，这个参数是用来告诉你在当前的系统平台下，是否对文件名大小写敏感 |
  | ---------------------- | ---- | ------------------------------------------------------------ |
  | lower_case_table_names | 0    | 默认为0，0-大小写敏感，1-大小写不敏感，2-表名存储为给定的大小写，比较时转成小写（实践发现2无用） |

  ```
  修改lower_case_table_names：
  	1. linux centos7版本下修改/etc/my.cnf文件增加
  		[mysqld]
  		lower_case_table_names=1
  	2. 重启数据库服务
  	3. 删除原来的数据库
  		drop database 数据库名 或者 Navicat for Mysql手动删除
  		可能会出现在客户端无法删除数据库的情况，那么只能在服务端删除，进入/var/lib/mysql，手动删除对		应的数据库
  	4. 重新导入数据库数据（实现需要备份），表名存储为给定的大小写（大小写不敏感生效）
  ```

- 修改时区

  ```
  进入mysql数据库，查看当前时区show variables like '%time_zone%';
  ```

  +---------------------------------+-------------------+
  | Variable_name                   | Value             |
  +---------------------------------+-------------------+
  | connect_timeout                 | 10                |
  | datetime_format                 | %Y-%m-%d %H:%i:%s |
  | delayed_insert_timeout          | 300               |
  | explicit_defaults_for_timestamp | OFF               |
  | innodb_flush_log_at_timeout     | 1                 |
  | innodb_lock_wait_timeout        | 50                |
  | innodb_rollback_on_timeout      | OFF               |
  | interactive_timeout             | 28800             |
  | lc_time_names                   | en_US             |
  | lock_wait_timeout               | 31536000          |
  | net_read_timeout                | 30                |
  | net_write_timeout               | 60                |
  | rpl_stop_slave_timeout          | 31536000          |
  | slave_net_timeout               | 3600              |
  | system_time_zone                | UTC               |
  | time_format                     | %H:%i:%s          |
  | time_zone                       | SYSTEM            |
  | timed_mutexes                   | OFF               |
  | timestamp                       | 1577937811.225014 |
  | wait_timeout                    | 28800             |
  +---------------------------------+-------------------+

```
其中system_time_zone=UTC，说明是标准时间，要改成东八区(China  Standard  Time,CST)，进入/etc/mysql/my.cnf,在[mysqld]下增加default-time-zone='+08:00'
```

- 彻底解决中文乱码

  ```
  init_connect='SET collation_connection = utf8_unicode_ci' 
  
  init_connect='SET NAMES utf8'
  
  character-set-server=utf8  
  
  collation-server=utf8_unicode_ci
  
  skip-character-set-client-handshake
  ```

- 备份数据库

  ```
  将数据库导出为sql文件
  	mysqldump -u 用户名 -p 数据库名 > 存放路径
  ```

- mysql各版本差异

  ```
  mysql 5.6及以下版本，默认支持主键长度为767bytes
  ```

  

