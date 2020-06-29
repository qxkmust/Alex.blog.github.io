# Docker Jenkins 部署

1. docker pull jenkins,拉取官网最新版本镜像

2. docker run -id --name=my_jenkins -p 映射到宿主机端口:容器默认8080 -v 宿主机挂载目录:/var/jenkins_home -v  /etc/localtime:/etc/localtime --restart=always 镜像id

   ```
   发现jenkins容器没有运行起来；
   查看日志docker logs my_jenkins:
   touch: cannot touch ‘/var/jenkins_home/copy_reference_file.log’: Permission denied
   Can not write to /var/jenkins_home/copy_reference_file.log. Wrong volume permissions?
   
   原因：
   /var/jenkins_home目录的拥有者未指定为jenkins（只有资源拥有者才有访问权限）
   
   解决：
   首先，查看jenkins容器拥有者的uid，
   使用命令：docker run -ti --rm --entrypoint="/bin/bash" 容器ID -c "whoami && id"
   将挂载到宿主机的目录的拥有者赋值给uid
   使用命令：chown -R uid 宿主机挂载目录
   最后，重新启动容器
   ```

3. 远程访问jenkins服务地址,并按提示安装插件

