# MongoDB的使用

### 安装

```
步骤一：
	①登录官网https://www.mongodb.com/download-center/community，下载相应版本，这里使用mongodb-linux-x86_64-rhel70-4.0.18.tgz
	②上传到hadoop02(192.168.8.105)的/opt目录，解压到当前目录/mongodb-4.0.18下
步骤二：配置
	①mongodb默认不写日志（只是打印到控制台），需要手动指定
		在/opt/mongodb-4.0.18下新建/logs目录，在里面增加mongodb.log日志文件
	②为mongodb增加db数据库文件存放目录（默认也是/data/db）
		在/opt/mongodb-4.0.18下新建/data/db目录
	③为mongodb指定运行配置文件mongo.conf
		在/opt/mongodb-4.0.18/bin下创建mongo.conf文件，编辑：
            dbpath = /opt/mongodb-4.0.18/data/db	#指定数据库文件存放目录
            logpath = /opt/mongodb-4.0.18/logs/mongodb.log	#指定日志文件
            port = 27017	#默认服务端口
            bind_ip_all = true  #开启远程连接
            fork = true		#以守护程序的方式启用，即可以在后台运行
步骤三：为mongodb创建连接认证信息
	mongodb默认是无密登录，所有操作都是root权限，生产中一定会进行安全认证
	①登录mongo客户端
	②在admin数据库中创建管理员用户
		①use admin
		②db.createUser({user:"root",pwd:"123456",roles:[{role:"root",db:"admin"}]})
	③使用navcat premium客户端连接mongodb服务端时，填写用户名root，密码123456
	注意：mongodb4版本认证密码至少6位
步骤四：数据库认证
	mongodb比较特殊，可以单独为每个数据库配置认证信息
	①使用步骤三中的认证信息登录mongo客户端
	②切换到需要认证的数据库
	③为数据库添加用户和角色
		db.createUser({user:"用户名",pwd:"密码",roles:[{role:"dbOwner",db:"数据库名"}]})
	注意：建议每个数据库都有唯一的用户名
步骤五：开启认证
	在/opt/mongodb-4.0.18/bin下创建mongo.conf文件，添加：
	auth = true    #开启认证
步骤六：设置开机自启动
	①编辑开机启动脚本/etc/init.d/mongodb：
```

```
#!/bin/sh
#
#mongod - Startup script for mongod
#
# chkconfig: - 85 15
# description: Mongodb database.
# processname: mongod
# Source function library
 
. /etc/rc.d/init.d/functions
# things from mongod.conf get there by mongod reading it
# OPTIONS
OPTIONS=" --config /opt/mongodb-4.0.18/bin/mongo.conf &"
#mongod
mongod="/opt/mongodb-4.0.18/bin/mongod"
lockfile=/var/lock/subsys/mongod
start()
{
  echo -n $"Starting mongod: "
  daemon $mongod $OPTIONS
  RETVAL=$?
  echo
  [ $RETVAL -eq 0 ] && touch $lockfile
}
 
stop()
{
  echo -n $"Stopping mongod: "
  killproc $mongod -QUIT
  RETVAL=$?
  echo
  [ $RETVAL -eq 0 ] && rm -f $lockfile
}
 
restart () {
        stop
        start
}
ulimit -n 12000
RETVAL=0
 
case "$1" in
  start)
    start
    ;;
  stop)
    stop
    ;;
  restart|reload|force-reload)
    restart
    ;;
  condrestart)
    [ -f $lockfile ] && restart || :
    ;;
  status)
    status $mongod
    RETVAL=$?
    ;;
  *)
    echo "Usage: $0 {start|stop|status|restart|reload|force-reload|condrestart}"
    RETVAL=1
esac
exit $RETVAL
```

```
②授权：
	chmod 777 /etc/init.d/mongodb
③将mongodb服务被chkconfig管理
	chkconfig --add mongodb
④开机启动
	chkconfig mongodb on
	
步骤四：启动/关闭
	启动命令：
		/opt/mongodb-4.0.18/bin/mongod --config /opt/mongodb-4.0.18/bin/mongo.conf
	关闭命令：
		/opt/mongodb-4.0.18/bin/mongo admin --eval "db.shutdownServer()"
	运行mongo客户端：
		/opt/mongodb-4.0.18/bin/mongo 【集合】 -u alex -p alex123
	
步骤七：编写启动/关闭脚本/usr/local/bin/mongo-shell：
		#!/bin/bash
        #启动/关闭 mongodb服务的脚本

        case $1 in
        "start"){
                echo "####### 开启mongod服务 #######"
                /opt/mongodb-4.0.18/bin/mongod -u root -p 123456 --config /opt/mongodb-4.0.18/bin/mongo.conf
        };;
        "stop"){

                echo "####### 关闭mongod服务 #######"
                /opt/mongodb-4.0.18/bin/mongo admin -u root -p 123456 --eval "db.shutdownServer()"
        };;
        "restart"){
                echo "####### 重启mongod服务 #######"
                /opt/mongodb-4.0.18/bin/mongo admin --eval "db.shutdownServer()"
                /opt/mongodb-4.0.18/bin/mongod --config /opt/mongodb-4.0.18/bin/mongo.conf
        }
 esac
```

### 常用术语

| SQL术语/概念 | MongoDB术语/概念 | 解释/说明                           |
| ------------ | ---------------- | ----------------------------------- |
| database     | database         | 数据库                              |
| table        | collection       | 数据库表/集合                       |
| row          | document         | 数据记录行/文档                     |
| column       | field            | 数据字段/域                         |
| index        | index            | 索引                                |
| table joins  | 不支持           | 表连接                              |
| primary key  | primary key      | 主键,MongoDB自动将_id字段设置为主键 |

### 常用数据类型

| 数据类型           | 描述                                                         |
| ------------------ | ------------------------------------------------------------ |
| String             | 字符串。存储数据常用的数据类型。在 MongoDB 中，UTF-8 编码的字符串才是合法的。 |
| Integer            | 整型数值。用于存储数值。根据你所采用的服务器，可分为 32 位或 64 位。 |
| Boolean            | 布尔值。用于存储布尔值（真/假）。                            |
| Double             | 双精度浮点值。用于存储浮点值。                               |
| Min/Max keys       | 将一个值与 BSON（二进制的 JSON）元素的最低值和最高值相对比。 |
| Arrays             | 用于将数组或列表或多个值存储为一个键。                       |
| Timestamp          | 时间戳。记录文档修改或添加的具体时间。                       |
| Object             | 用于内嵌文档。                                               |
| Null               | 用于创建空值。                                               |
| Symbol             | 符号。该数据类型基本上等同于字符串类型，但不同的是，它一般用于采用特殊符号类型的语言。 |
| Date               | 日期时间。用 UNIX 时间格式来存储当前日期或时间。你可以指定自己的日期时间：创建 Date 对象，传入年月日信息。 |
| Object ID          | 对象 ID。用于创建文档的 ID。                                 |
| Binary Data        | 二进制数据。用于存储二进制数据。                             |
| Code               | 代码类型。用于在文档中存储 JavaScript 代码。                 |
| Regular expression | 正则表达式类型。用于存储正则表达式。                         |

### 常用操作

权限认证

```
查看用户
【语法】db.system.users.find({})，查看全部用户
【语法】db.getUsers()，查看当前数据库用户
新增用户
【语法】db.createUser({user:"用户名",pwd:"密码",roles:[{role:"角色名",db:"数据库"}]})
		角色名包括：
            Read：允许用户读取指定数据库
            readWrite：允许用户读写指定数据库
（生产中常用）dbOwner: 允许用户在指定数据进行DDL和DML操作
            dbAdmin：允许用户在指定数据库中执行管理函数，如索引创建、删除，查看统计或访问system.profile
            userAdmin：允许用户向system.users集合写入，可以找指定数据库里创建、删除和管理用户
            clusterAdmin：只在admin数据库中可用，赋予用户所有分片和复制集相关函数的管理权限。
            readAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的读权限
            readWriteAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的读写权限
            userAdminAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的userAdmin权限
            dbAdminAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的dbAdmin权限。
            root：只在admin数据库中可用。超级账号，超级权限
删除用户
	【语法】db.dropUser("用户名")
修改用户密码
	【语法】 db.updateUser("用户名",{pwd:"新密码"})
为已存在用户增加角色
	【语法】db.grantRolesToUser("用户名",[{role:"角色名",db:"数据库"}])
```

数据库

```
show dbs;查看所有数据库
db;当前窗口的数据库
use 数据库名;切换数据库
db.数据库名.dropDatabase();删除当前数据库
```

集合

```
查看当前数据库下所有集合
	show collections;
统计集合中的元素数量
	db.集合名.count();
删除集合
	db.集合名.drop();
手动创建集合
	db.createCollection("集合名");
通过插入操作自动创建集合
	db.集合名.insert({属性名:属性值});
```

查询操作

```
查询集合中所有数据
	db.集合名.find();
条件查询：
	1.按照属性值查询
		db.集合名.find({属性名:属性值});
	2.比较运算符查询：
	【语法】：db.集合名.find({属性名:{筛选条件:属性值}});根据筛选条件查询
    比较运算符包括：
        $gt(大于)
        $lt(小于)
        $lte(不大于)
        $gte(不小于)
        $ne(等于)
	如：查找年龄大于18岁的学生信息：
		db.stu.find({age:{$gt:18}})
	3.包含
	【语法】：db.集合名.find({属性名:{包含运算符:[属性值1,属性值2,...]}})
	包含运算符：
		$all(全部包含，要求属性字段必须是数组)
		$in(包含)
		$nin(不包含)
		如：查找年龄在[18,19,20]内的学生
			db.stu.find({age:{$in:[18,19,20]}})
	4.分页
	【语法】db.集合名.find().skip(0).limit(n);从第一条开始，查n条
	5.排序
	db.集合名.find().sort({属性名:排序规则})，排序规则：1（升序），-1（降序）
	如：按照学生年龄降序
		db.stu.find().sort({age:-1})
```

删除操作

```
按照属性值删除
	【语法】:db.集合名.remove({属性名:属性值})
条件删除
	【语法】:db.集合名.remove({属性名:{运算符:属性值}})
	运算符：
		 	$gt(大于)
            $lt(小于)
            $lte(不大于)
            $gte(不小于)
            $ne(等于)
            $all(全部包含，要求属性字段必须是数组)
            $in(包含)
            $nin(不包含)
```

更新操作

```
【语法】：db.集合名.update(旧数据，新数据，如果没有旧数据是否新增，是否批量更新)
是否新增：0（不新增），1（新增）
是否批量更新：0（只改第一条），1（批量更新）
如：将成绩为100分改成98分
	db.stu.update({score:100},{score:98},0,1)
```

索引

```
默认索引
	_id,每条数据的_id都是唯一的
手动增加索引
	db.索引名.ensureIndex({属性名:排序规则})，排序规则：1（升序），-1（降序）
删除索引
	db.索引名.dropIndex({属性名:排序规则})，排序规则：1（升序），-1（降序）
```

投影

```
显示一个或多个属性
	db.集合名.find({},{属性名:1},{属性名:1})，排序规则：1（升序），-1（降序）
```

### Mongodb与ES对比

|                      | Mongodb    | ES                             |
| -------------------- | ---------- | ------------------------------ |
| 使用场景             | 文档写操作 | 读操作多的场景：检索、数据分析 |
| 存储格式             | json       | json                           |
| CRUD                 | 支持       | 支持                           |
| 数据规模             | TB         | GB                             |
| 事务                 | 不支持     | 不支持                         |
| 聚合                 | 支持       | 支持                           |
| 表的数据结构动态改变 | 支持       | 不支持                         |
| 写入速度             | 快         | 慢（有1s延时）                 |

