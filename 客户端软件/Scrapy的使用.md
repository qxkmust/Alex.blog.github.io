# Scrapy的使用

### 简介

```
Scrapy是Python开发的一个快速、高层次的屏幕抓取和web抓取框架，用于抓取web站点并从页面中提取结构化的数据。Scrapy用途广泛，可以用于数据挖掘、监测和自动化测试。
```

### 安装

```
以下针对python3.8.3版本有效，其它版本未验证
安装步骤：
步骤一、准备python环境
	这里使用python3.8.3版本，配置系统环境变量%PYTHON_HOME%和%PYTHON_HOME%\Scripts
	
步骤二、网络安装wheel
	在cmd命令窗口，使用命令：
		pip install wheel，默认适配当前python版本进行安装
	注意：如果出现以下提示：“WARNING: You are using pip version 19.2.3, however version 20.1.1 is available.
You should consider upgrading via the 'python -m pip install --upgrade pip' command.”
	解决：自动下载的wheel版本不匹配，使用以下命令解决：
		python -m pip install --upgrade pip
		
步骤三、本地安装lxml
	进入网站https://www.lfd.uci.edu/~gohlke/pythonlibs/#lxml，下载lxml-4.5.2-cp38-cp38-win_amd64.whl到D:\Python\3.8.3\Scrapy目录下
	使用cmd安装：pip install lxml-4.5.2-cp38-cp38-win_amd64.whl
	
步骤四、本地安装pyWin32
	进入网站https://www.lfd.uci.edu/~gohlke/pythonlibs/#pywin32，下载pywin32-228-cp38-cp38-win_amd64.whl
	使用cmd安装：pip install pywin32-228-cp38-cp38-win_amd64.whl
	
步骤五、本地安装Twisted
	进入网站https://www.lfd.uci.edu/~gohlke/pythonlibs/#twisted，下载Twisted-20.3.0-cp38-cp38-win_amd64.whl
	使用cmd安装：pip install Twisted-20.3.0-cp38-cp38-win_amd64.whl
	
步骤六、网络安装Scrapy
	安装以上依赖后，开始安装scrapy，使用命令：pip install scrapy
```

### 架构

![scrapy爬虫框架架构](D:\Git\MyRepository\Alex.blog.github.io\Images\scrapy爬虫框架架构.png)

### 案例

```
需求：
	指定搜索栏的关键字，爬取百度图片，并下载到本地
实现：
	①创建一个scrapy项目
		cmd命令进入D:\Python\3.8.3\MyProject目录下,使用命令创建项目demo01:
			scrapy startproject demo01
			，发现scrapy自动创建demo01工程目录
	②定义item容器
		编辑D:\Python\3.8.3\MyProject\demo01\demo01\items.py
	③编写爬虫
		编辑D:\Python\3.8.3\MyProject\demo01\demo01\spiders\Demo01Spider.py
	④存储内容
		
```

### 常见问题

##### 爬虫出现Forbidden by robots.txt

```
解决：
	关闭scrapy自带的ROBOTSTXT_OBEY功能，在setting.py找到这个变量，设置为False即可解决。
补充：
	Robots协议是Web站点和搜索引擎爬虫交互的一种方式，Robots.txt是存放在站点根目录下的一个纯文本文件。该文件可以指定搜索引擎爬虫只抓取指定的内容，或者是禁止搜索引擎爬虫抓取网站的部分或全部内容。当一个搜索引擎爬虫访问一个站点时，它会首先检查该站点根目录下是否存在robots.txt，如果存在，搜索引擎爬虫就会按照该文件中的内容来确定访问的范围；如果该文件不存在，那么搜索引擎爬虫就沿着链接抓取。
```

