# SecureCRT的使用技巧

### 解决中文乱码

##### 临时解决

```
会话级的编码设置方式：

选项-->会话选项-->终端-->外观-->字符编码格式-->选择utf8编码

新开一个会话就要重新设置
```

##### 永久解决

```
设置会话的默认初始化选项：

需要找到SecureCRT的配置文件：D:\SecureCRT\Config\Sessions\Default.ini

将S:”Output Transformer Name”=Default修改为S:”Output Transformer Name”=UTF-8

之后在-->会话选项再次修改编码为utf8

最后，重启SecureCRT
```

### 关闭提示音

```
修改Default.ini文件：
	D:"Audio Bell"=00000001 改为 D:"Audio Bell"=00000000
```

### 修改颜色

```
选项 -> 全局选项 -> ANSI颜色 -> 8个标准颜色依次对应背景色、压缩包、可执行文件、命令符、目录、图片、软连接、普通文本
```

