```markdown
# 记录aliyun ECS问题-01
## 解决阿里云服务器centos被远程开启定时任务，导致cpu和内存占满的问题

# 问题复现
* *docker服务访问缓慢，甚至访问不到（不知道为什么会被linux给杀掉）*

# 排查过程
* *使用远程工具连接服务器，使用htop工具查看cpu和内存占用情况，发现几乎接近100%*

# 一般解决办法
* *使用crontab -l查看定时任务*
* *使用nslookup解析域名/ip*
* *最后在ECS安全组拦截该ip，如果拦截无效，可以使用crontab -r删除定时任务*

# 感谢帖子
[^1]
[url](http://www.yunzhongmoke.com/articles/2019/11/21/1574302653004.html)
![Image](http://img1.cache.netease.com/catchpic/C/CB/CBA385C092F9E982A998F38D91A43D98.jpg)

# 终极解决办法
[^2]
[url](https://blog.csdn.net/qq_33376750/article/details/77354423)
![Image](http://img1.cache.netease.com/catchpic/C/CB/CBA385C092F9E982A998F38D91A43D98.jpg)

```