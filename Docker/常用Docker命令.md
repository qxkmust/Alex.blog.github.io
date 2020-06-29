# 常用Docker命令

- 从docker hub查找镜像

  docker search 镜像名

- 拉取镜像

  docker pull 镜像名(docker.io后的全名)

- 备份镜像(-o 指定输出到文件)

  docker save -o 压缩文件名.tar 要备份的镜像名:版本号 

- 恢复镜像(-i 指定镜像文件)

  docker load -i 压缩文件名.tar

- 查看日志(-f 追踪日志实时显示，--tail=100 最后100行)

  docker -f --tail=100 容器名

- 查看所有docker容器

  docker ps -a 

- 查看容器挂载目录

  docker inspect --format='{{.Mounts}}' 容器名

- 创建（-d）并运行（-i）守护docker容器

  docker run -id --name=名字 -p 宿主端口:docker端口 -v 宿主路径1:docker路径1  -v 宿主路径2:docker路径2  --restart=always 镜像名:版本号

  ```
  -id 创建守护容器（一创建就会启动）
  -p 指定端口映射
  -v 挂载路径/文件映射
  --name 指定名称
  --restart=always 跟随docker服务运行（docker服务重启后自动运行）
  ```

  

- 容器发布成镜像

  docker commit 要发布的容器名 新的镜像名:版本号

- 登录容器,进入命令行终端

  docker exec -it 容器名 /bin/bash

- 退出容器

  exit

- 启动/停止容器

  docker start/stop 容器名

- 拷贝宿主机中文件到容器

  docker cp 要拷贝的宿主机文件或目录 容器名称:容器文件或目录

- 删除容器

  docker rm 容器名/容器id

- 删除镜像

  docker rmi 镜像名:版本号

- 本地镜像发布到阿里云镜像仓库

  ```
  1. docker login --username=aininala@sina.cn 阿里云registry地址(registry.cn-shenzhen.aliyuncs.com)
  2. docker tag 镜像id 阿里云registry地址/命名空间/镜像名:版本号
  3. docker push 阿里云registry地址/命名空间/镜像名:版本号
  ```

  

- 从阿里云镜像仓库拉取

  docker pull 阿里云registry地址/命名空间/镜像名:版本号

- 难点-使用Dockerfile构建镜像

  ```
  docker build [-f 指定Dockerfile所在路径与文件名] -t 生成的镜像名:标签名 . 
  注意后边的 空格 和点 . 不要省略, . 表示当前目录
  -f 指定Dockerfile文件所在路径与文件名。如果未指定 -f 值，则找当前目录下名为 Dockerfile 的构建
  文件
  ```

  