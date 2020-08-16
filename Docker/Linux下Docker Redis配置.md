# Linux下Docker Redis的配置

#### 1.如何部署redis Docker容器

-  在宿主机上新建redis容器的数据持久化文件以及配置文件：xx/xx/data 和 xx/xx/redis.conf（从官网下载 http://download.redis.io/redis-stable/redis.conf ）

  ```
  redis.conf 配置(从上至下依次)：
  
  bind 127.0.0.1 #注释掉这部分，这是限制redis只能本地访问
  
  protected-mode no #默认yes，开启保护模式，限制为本地访问
  
  daemonize no#默认no，改为yes意为以守护进程方式启动，可后台运行，除非kill进程，改为yes会使配置文件方式启动redis失败
  
  databases 16 #数据库个数（可选），我修改了这个只是查看是否生效。。
  
  dir  ./ #输入本地redis数据库存放文件夹（可选）
  
  appendonly yes #redis持久化（可选）
  ```

  

- docker run --name=容器名 -p 映射端口:6379 -v xx/xx/data:/data -v xx/xx/redis.conf:/etc/redis/redis.conf -id 镜像ID redis-server /etc/redis/redis.conf

  ```
  -p 宿主机端口:容器服务端口
  -v 宿主机挂载文件（目录）:容器文件（目录）
  redis-server /etc/redis/redis.conf 运行指定的配置文件
  ```