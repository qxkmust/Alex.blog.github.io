# Docker Mysql容器运行太占内存怎么办?

*最近发现1G内存的Aliyun服务器运行Mysql Docker服务被占用大量内存(即使PV很少的情况下),大概500M.*

<u>官方的Mysql大小也就200M,表的数量也就10个,数据量也非常少,为什么表空间会这么大呢?</u>

原因就在于: performance_schema默认打开了,什么是performance_schema,以下介绍:

![1574859639979](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1574859639979.png)

说了这么多,了解到performance_schema开启后是非常占用内存的,于是对于内存很宝贵的云服务器,可以考虑在mysql的配置文件/etc/mysql/my.cnf中增加配置项:**[mysqld] performance_schema=off**