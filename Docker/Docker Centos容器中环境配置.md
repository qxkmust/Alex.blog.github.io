# Docker Centos容器中环境配置

docker容器启动后是默认配置，需要修改一些基本配置：

- 默认没有指定中文编码

  在/etc/locale.conf中将 LANG=en_US.UTF-8改为LANG=zh_CN.UTF-8，重启服务器 

