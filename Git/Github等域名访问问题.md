# Github等域名访问问题

Github、mvnrepository等域名经常访问缓慢或者无法访问

解决：

步骤一：查找最新的域名ip

​	登录ipaddress.com，查找域名对应的最新的ip（可能有多个）

步骤二：在本机C:\Windows\System32\drivers\etc\hosts文件中增加域名/ip绑定

​	先将hosts文件复制到别处进行修改，之后再替换原文件

步骤三：刷新DNS缓存，使立即生效

​	在cmd命令窗口刷新dns，ipconfig /flushdns