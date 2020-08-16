# Linux下Redis的常见问题

#### 1.redis在linux上的安装和使用

```
一：安装
	①官网https://redis.io/下载最新的tar.gz安装包，通过ftp工具上传到linux终端目录/opt（/opt是专门存放用户的程序）
	②解压，tar -zxvf redis.tar.gz文件 
	③make命令编译
		一般如果是虚拟机编译，可能会报错：
		①需要gcc编译环境
			安装gcc，通过yum -y install gcc
        ②重新make，可能继续报错：文件或目录jemalloc/jemalloc.h
			运行make distclean，再次make
		③安装redis-server和redis-cli
			进入redis安装目录/src，make install
二：设置开启自启动
    ①修改配置文件redis.conf，daemonize=yes
    ②在linux/etc/init.d/路径下添加redis自启动脚本
        ①redis自带了自启动脚本文件，/opt/redis-5.0.8/utils/redis_init_script，拷贝到/etc/init.d下，命名为redis，cp redis_init_script /etc/init.d/redis
        ②更改权限，因为系统会读取和执行脚本文件，需要保证redis脚本文件有读r和执行x的权限
            chmod 755 redis
    ③加入到系统的开机启动
        chkconfig redis on
三：redis的使用
    ①常用命令（参考http://redisdoc.com/）
        ①切换库，select 库号，默认16个数据库0~15，默认使用0号库
        ②dbsize查看当前书库的key的数量
        ③flushdb清空当前库
        ④flushall，通杀全部库
        ⑤save，立刻将当前操作备份到dump.rdb
	②启动redis服务，redis-server redis.conf
	③关闭redis服务
		①redis-cli shutdown
		②也可以在redis-cli连接中使用shutdown关闭
	④开启redis远程访问，配置redis.conf
		①注释#bind=127.0.0.1
		②关闭保护模式，开启远程访问，protected-mode=no
```

#### 2.redis开启远程连接

```
修改reids配置文件redis.conf：
    ①注释bind=127.0.0.1
    ②关闭保护机制（默认是yes），protect-mode=no
    ③关闭linux的防火墙设置
    	①查看防火墙状态，systemctl status firewalld
    	②关闭
    		①临时关闭，systemctl stop firewalld
    		②永久关闭，systemctl disable firewalld
```

#### 3.redis的基本数据类型（详细参考redisdoc.com）

| 类型   | 描述     | 特点              | 使用场景                                                     |
| ------ | -------- | ----------------- | ------------------------------------------------------------ |
| String | 字符串   | key→value         | 存放黑/白名单                                                |
| List   | 列表     | 有序              | 常用来存放最新消息的排行<br>比如：推荐系统项目根据用户评分进行实时推荐时<br>web业务系统将用户的评分信息<br>[userId, productId, rate, timestamp]<br>实时存放到redis 列表中，以供推荐系统根据该用户<br>的最新评分给出实时推荐商品 |
| Hash   | 哈希     | [key,field]→value | 存储用户个人信息<br>【用户ID，用户名，XXX】<br>【用户ID，等级，XXX】<br/>【用户ID，星座，XXX】 |
| Set    | 集合     | 无序不重复        | 在朋友圈、微博应用中，使用集合存放用户所有关注的人<br>，所有的粉丝 |
| zSet   | 有序集合 | 有序不重复        | 排行榜                                                       |

```
①键-key
    ①key的模糊查询
        keys hell?，可以匹配hello
        keys he*o，可以匹配hello，helo
        keys h[ae]llo，可以匹配hallo和hello
②字符串-String
	①get/set/del/append/strlen
	②incr/decr/incrby/decrby 只有数字才能进行加减
	③getrange（获取角标之间的字符串，用法：getrange key 开始下标 结束下标）/setrange（设置指定角标的字符串，用法：setrange key 开始下标 结束下标）
	④setex（set expire time，指定时间后key被删除），用法setex key 秒数 value
	⑤setnx（set if not exists，存在将不做任何操作），用法setnx key value
	⑥mget/mset（批量获取/设置键值）
	⑦msetnx（批量设置键值，全部成功返回1）
③列表-List
	①lpush（类似每次在栈底插入元素）/rpush/lrange（获取指定角标范围的元素，用法lrange key 开始下标 结束下标）
	②lpop（类似将栈底的元素出栈）/rpop（类似将栈顶的元素出栈）
	③lindex（按照角标获取元素）
	④llen（获取列表长度）
④集合-Set（元素不可重复）
	①sadd（添加元素）/smembers（获取所有元素）
	②scard（获取集合中的元素个数）
	③srem（删除元素）
	④srandmember key 数字
		数字i为正数且小于元素总数n，随机返回i个不重复的元素
			用处：随机抽奖
		数字i为负数，随机返回i个可能重复的元素
	⑤spop key（随机出栈）
⑤哈希-Hash
	①hget（根据key和field获取单个的值）/hmget(获取多个值，用法：hmget key field1 field2...)/hset（设置单个hash键值对，用法：hset key field value）/hmset（设置多个hash键值对，hmset key field1 value1 field2 value2 ...）
	②hdel(根据key和field删除元素)
	③hexists（根据key和field判断元素是否存在，存在返回1，不存在返回0）
⑥有序集合-Zset
	与set的区别：值中增加了score属性
	①zadd（添加一个或多个元素，用法：zadd key score1 v1 score2 v2...）
	②zrange（根据key，开始下标，结束下标获取多个元素，用法：zrange key 开始下标 结束下标）
	③zcount（统计两个score之间的元素数，用法：zcount key score1 score2）
	④zrank（排序）
```

#### 4.redis.conf配置详细

```
	①缓存过期策略参数：maxmemory-policy
		①volatile-lru，对于设置了过期时间的键使用LRU算法移除
		②allkeys-lru，使用LRU算法移除key
		①volatile-random，随机移除key
		④allkeys-random，随机移除key
		⑤volatile-ttl，移除ttl最小（最近要过期）的key
		⑥noeviction，不移除，写操作直接报错（生产不能用）
        注意：
        ①从缓存类型来说，其中名称中带volatile的策略确定了被淘汰的缓存仅从设置了过期时间的key中选择，如果没有任何key被设置过期时间，那么Volatile-lru，Volatile-random，Volatile-lfu表现得就分别和allkeys-lru，allkeys-random，allkey-lfu算法相同。
		②如果缓存有明显的热点分布，那么应该选择lru类型算法，如果缓存被访问几率相同，那么应该选择随机淘汰算法。
    ②限制redis由指定的ip访问，注释掉后可以支持本地和远程（配合protected-mode=no）的ip访问，bind 127.0.0.1 
    ③开启保护模式，限制为本地访问，默认yes，protected-mode no 
    ④默认no，改为yes意为以守护进程方式启动，可后台运行，除非kill进程，改为yes会使配置文件方式启动redis失败，daemonize no
    ⑤数据库个数（可选），databases 16 
    ⑥redis持久化（默认no关闭），appendonly yes 
    ⑦timeout 秒数，当客户端闲置多长时间后关闭连接，0-表示关闭该功能
    ⑧loglebel，日志记录级别，由低到高：debug、verbose（默认）、notice、warning
    ⑨logfile ""，指定日志文件的存放路径，如果redis以守护进程的方式运行，日志不会写到控制台，如果非要看可以指定到日志文件中
    ⑨pidfile pid文件全路径名，redis如果以守护进程方式运行，redis会把pid写入到/var/run/redis.pid中，通过pidfile可以指定这个路径
    ⑩dir 指定dump.rdb文件的路径（默认是/）
    11.save 秒数 写操作次数，redis以rdb文件进行数据持久化，默认满足三种任意一种均会生成dump.rdb文件
    	save 900 1 （15分钟内1次写操作）
    	save 600 10（10分钟内10次写操作）
    	save 60 10000（1分钟内10000次写操作）
    	save "" (关闭rdb数据持久化)
    12.appendonly，开启aof持久化开关，默认关闭
    13.appendfilename，指定aof文件名，默认“appendonly.aof“，文件位置与第⑩dir设置的dump.rdb是同一个目录
    14.appendfsync，指定aof日志记录的方式
    	always 每次发生写操作数据立刻被记录到磁盘，数据完整性最好性能最差
    	everysec 默认，每秒记录一次，最多丢失一秒的数据
    	no 不记录
    15. auto-aof-rewrite-min-size,设置重写的基准值（64mb）
    16. auto-aof-rewrite-percentage，设置重写的基准值（默认100，即100%）
```

#### 5.redis持久化

```
一：RDB，是一种指定时间间隔内将内存中的数据（snapshot快照）写入到磁盘，以便在数据恢复时将快照文件读到内存中。
	①原理：redis单独创建（fork）一个子进程来进行持久化，数据首先被写入到一个临时的dump文件，最终在满足save配置的秒数后，这个临时文件会替换上次的dump文件。
	②优缺点：
		优点：主进程不参与io，性能很高；
		缺点：
			①数据备份可能不完整，如果dump还没有来得及替换旧的dump，就发生网络故障，最后一次dump的备份将全				部丢失
			②每次fork，内存中的数据被子进程克隆一份，比较耗费内存
	③用法：
		①开启RDB快照，redis配置文件redis.conf中默认提供三种，见4-11
		②可选，修改dump.rdb文件的存放路径，见4-10
		③重启redis服务，使配置生效
		④redis客户端连接服务，对key写操作，观察dump.rdb文件大小，du -b dump.rdb
二：AOF，是一种以日志的形式来记录每个写操作，将redis执行过的每个写指令追加到appendonly.aof文件中，redis启动的时候读取aof文件进行数据重构。
	 ①
	 ②优缺点：	
	 	优点：持久化fsync策略的粒度更细，数据一致性高
	 	缺点：
	 		①细粒度的持久化策略的副作用就是时间开销大于RDB
	 		②aof日志文件大于RDB
	 ③rewrite机制
	 	①解决aof文件持续增长过大（当aof文件大于64mb，且超出上次重写后大小的一倍），会fork出一条新进程，将每个写操作命令放到临时文件中
三：对比：
	①redis服务启动时，载入数据集的优先级 aof > rdb
	②redis默认关闭aof的日志记录功能（appendonly no），rdb快照方式默认开启（有三种save机制）
	③aof的持久化策略粒度更细，数据一致性高，但是通过aof进行数据恢复比较耗时；rdb的数据实时性低，可能会丢失最后一次保存的dump.rdb。
```

#### 6.redis的事务

```
一：用法：	
	①开启事务，MULTI
	②入队，一系列redis读写操作
	③提交事务，EXEC
	④取消事务（当前已经加入事务的队列中的命令不会被执行），DISCARD
	⑤监控锁WATCH，类似于乐观锁（由于redis的事务不具有原子性，监控锁可以保证事务原子性）
		用法：
		①客户端1开始监控key：WATCH [key1,key2] >> 开启事务MULTI >>有客户端2修改了key的值 >> 客户端1提交事务会报错
		②UNWATCH，取消对所有key的监控
二：特性：
	①”连坐受罚“，如果将某个操作命令加入事务的过程中发生错误（类似java编译出错），这个事务的所有操作都不会执行
	②“冤有头债有主”，如果加入到事务中的操作命令在执行时（类似运行时异常）出错，不会影响其他的操作的执行
	③弱一致性，从第③看出，Redis的事务不能保证原子性
```

#### 7.redis的消息订阅/发布

```
比消息中间件rabbitMQ,SMS,kafka差很多，不建议使用
用法：
	①开启两个或者多个redis客户端
	②客户端1一次性订阅多个，SUBSCRIBE news1,news2,news3
	③客户端2发布消息
			PUBLISH news1 good_news1
			PUBLISH news2 good_news2
			PUBLISH news3 good_news3
```

#### 8.redis主从复制

```
	概念：redis主从复制，是指将一台redis（主节点）服务器的数据，复制到其他的redis服务器（从节点），数据的复制是单向的（只能由主到从），只有主节点能对数据进行写操作，从节点适合大量读操作
	原理：
		①从节点连接到主节点后会发送一个psync命令到主节点，主节点将所有的写操作指令放到临时快照dump.rdb文件，替换掉从节点的dump.rdb，从节点在后台进行全量复制，最终实现数据的同步
		②全量复制：从节点一次性将rdb文件加载到内存，在后台执行所有的写操作
		③增量复制：主节点不间断的将收集到的写操作命令依次发送给从节点，完成主从数据同步
	分类：
		①一主多从，一个主节点对应多个从节点
			用法：手动为从节点设置主从关系，replicaof 主节点ip 主节点端口
		②薪火相传，一个从节点也可是是另一个从节点的主节点
			用法：手动将多个从节点设置主从关系
		③反客为主，断开当前的连接关系，不再与之前主节点有关系，从节点角色变为主节点
			用法：手动在从节点使用命令：replicaof no one
		④哨兵模式（反客为主的自动版，后台监控到主机故障后会让从机投票选出新的主节点）	
			用法：在主节点的redis路径/sentinel.conf中增加sentinel monitor 主节点ip 主节点port 数字（从节点投票数）
		
	一主多从模式操作：
        ①配置所有的redis服务器
            1.配置redis_6379服务器(虚拟机ip192.168.8.103)，参考part1和part2，类似的，再配置redis_6380服务器(虚拟机ip192.168.8.101)和redis_6381服务器(虚拟机ip192.168.8.102)，注意在配置pidfile和port时三者有所不同
            2.配置完后验证，保证三者通过redis客户端能互相连接
        ②从节点连接到自身的redis服务器后，使用命令replicaof 主节点ip 主节点端口，查看节点的主从关系，info replication
        ③验证
            ①在从节点查看是否已经全量复制了主节点的数据集，keys *
            ②对主节点进行写操作，从节点是否增量复制了主节点的数据集
		注意：
			①主从模式默认是读写分离，主节点可以读写，从节点只能读，如果要使从节点具有写权限，可以在从节点redis.conf配置中修改replica-read-only no（但是不建议）
			②如果主节点发生网络故障挂掉，可以手动将一个从节点切换为主节点（replicaof no one），之后再重启旧的主节点，并将旧的主节点设置为新的主节点的从节点（replicaof 新主节点ip 新主节点port）
				原因：如果网络故障后直接重启主节点，可能主节点还没有进行持久化，重启后直接将不完整的dump.rdb文件传给从节点，导致主节点和从节点都会丢失数据
			③如果从节点挂掉，重启后将失去与主节点的主从关系，需要重新设置为从节点
			④如果主节点挂掉，重启后依然还是主节点，与其它从节点的关系不变
			
	哨兵模式操作:
		①配置所有的redis服务器
            1.配置redis_6379服务器(虚拟机ip192.168.8.103)，参考part1和part2，类似的，再配置redis_6380服务器(虚拟机ip192.168.8.101)和redis_6381服务器(虚拟机ip192.168.8.102)，注意在配置pidfile和port时三者有所不同
            2.配置完后验证，保证三者通过redis客户端能互相连接
        ②从节点连接到自身的redis服务器后，使用命令replicaof 主节点ip 主节点端口，查看节点的主从关系，info replication
        ③配置redis_6379服务器redis目录/sentinel.conf，增加sentinel monitor 192.168.8.103 6379 1，意思是监控主节点的状态，如果主节点挂掉，立刻让从节点们进行投票选举，票数大于1的重新作为主节点
        ④手动shutdown redis_6379服务器，观察log/log.log日志中打印从节点投票选举主节点的过程
```

#### 9缓存穿透、缓存雪崩、缓存击穿

```
1）缓存穿透
	是指查询一个一定不存在的数据。由于缓存命不中时会去查询数据库，查不到数据则不写入缓存，这将导致这个不存在的数据每次请求都要到数据库去查询，造成缓存穿透。
解决方案：
① 是将空对象也缓存起来，并给它设置一个很短的过期时间，最长不超过 5 分钟
② 采用布隆过滤器，将所有可能存在的数据哈希到一个足够大的 bitmap 中，一个一定不存在的数据会被这个 bitmap 拦截掉，从而避免了对底层存储系统的查询压力
2）缓存雪崩
	如果缓存集中在一段时间内失效，发生大量的缓存穿透，所有的查询都落在数据库上。
解决方案：
	尽量让失效的时间点不分布在同一个时间点
3）缓存击穿
	是指一个 key 非常热点，在不停的扛着大并发，当这个 key 在失效的瞬间，持续的大并发就穿破缓存，直接请求数据库，就像在一个屏障上凿开了一个洞。
解决方案：
可以设置 key 永不过期
```

