# JVM调优

### -Xmx -Xms -Xss

```
1. -Xmx  设置应用程序（不是JVM）内存可用大小 （ 如果程序要花很大内存的话，可以修改缺省配置，但是不能超过机器的内存），即最大可用Heap的大小。

2. -Xms  设置初始Heap的大小 （设置这个值启动性能会提高，也会受到机器内存的限制和最大Heap的限制）

很多情况下，一般-Xmx和-Xms 大小设置成一样大，因为不一样的话，在程序内存变化的情况下，每次垃圾回收后，都会重新分配内存。

3. -Xss 规定了每个线程堆栈的大小。一般情况下256K是足够了。影响了此进程中并发线程数大小。
```



### 性能监控

```
uptime命令
结果[系统时间 运行时间 终端连接数 1，5，15分钟内的系统平均负载]
```

![](https://github.com/qxkmust/Alex.blog.github.io/blob/blog/Images/UPTIME%E5%91%BD%E4%BB%A4.png?raw=true)

```
top命令（htop工具功能更强）
结果如图
```

![](https://github.com/qxkmust/Alex.blog.github.io/blob/blog/Images/top%E5%91%BD%E4%BB%A4.png?raw=true)

```
vmstat命令[vmstat 1 5 表示每秒采样一次，采样5次]
si:每秒从磁盘读入虚拟内存的大小，如果这个值大于0，表示物理内存不够用或者内存泄露了，要查找耗内存进程解决掉
so:每秒虚拟内存写入磁盘的大小，如果这个值大于0，同上
bi:块设备每秒接收的块数量，这里的块设备是指系统上所有的磁盘和其他块设备，默认块大小是1024byte
bo:块设备每秒发送的块数量，例如我们读取文件，bo就要大于0。bi和bo一般都要接近0，不然就是IO过于频繁
in:每秒cpu中断次数，越小越好
cs:每秒上下文切换次数，越小越好（调用系统函数或者线程切换会发生）
us:就是用户的进程获得了CPU资源以后，在用户态执行的时间（如果太高，可能死循环)
sy：用户进程获得了CPU资源以后，在内核态的执行时间（如果太高，表示系统调用时间长，例如IO操作频繁）
id:空闲 CPU时间，一般来说，id(空闲CPU使用率) + us(用户CPU使用率) + sy(系统CPU使用率) = 100%
wt:等待IO CPU时间。
```

![](https://github.com/qxkmust/Alex.blog.github.io/blob/blog/Images/vmstat%E5%91%BD%E4%BB%A4.png?raw=true)

```
pidstat命令（通过yum install sysstat工具提供）
```

![](https://github.com/qxkmust/Alex.blog.github.io/blob/blog/Images/pidstat%E5%91%BD%E4%BB%A4.png?raw=true)

### JAVA自带工具包

| 命令/工具                           | 说明                                                         | 举例                                          |
| ----------------------------------- | ------------------------------------------------------------ | --------------------------------------------- |
| jps                                 | 查看当前java进程的pid                                        | jps                                           |
| jstack pid >>输出到文件             | 打印当前线程堆栈信息到文件（线程间死锁、死循环、请求外部资源导致的长时间等待） | jstack 10001 >> stack.txt                     |
| jmap -dump:format=b,file=文件名 pid | 打印堆信息到文件                                             | jmap -dump:format=b,file=/home/dump.txt 10001 |
| JConsole工具                        | 图形化界面                                                   | 连接到需要监控的java进程                      |
| Visual VM                           | 比JConsole更加强大                                           |                                               |
| Arthas                              | 阿里开源的Java诊断工具，超牛逼                               | 使用方式：命令行交互方式                      |

通过命令只能生成执行中的jvm线程快照，一般用于排查死锁、死循环等线程还存活的情况，如果线程已经执行完了，则监控不到。

