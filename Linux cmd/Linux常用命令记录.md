# Linux常用命令记录

### **查找**或者 **文件搜索** 命令: 

- 推荐whereis搜索的是整个文件系统 或者which（只是path变量指定的路径下）

- find / -name 文件名

- ps -ef|grep 进程名

- netstat -nlp|grep 端口

- crontab

    - -l #查看系统定时任务
    - -e 编辑用户自定义定时器
    - -r  删除用户定时器

- grep  [OPTIONS] PATTERN [FILE...]

  ```
  -V, --version
  -G, --basic-regexp    BRE 模式，也是默认的模式
  -E, --extended-regexp ERE 模式
  -P, --perl-regexp      PRE 模式
  -F, --fixed-strings     指定的模式被解释为字符串
  -i 忽略大小写
  -o 只输出匹配到的部分(而不是整个行)
  -v 反向选择，即输出没有没有匹配的行
  -c 计算找到的符号行的次数
  -n 顺便输出行号
  ```

- ln -s 源目录/文件 目的目录/文件

  ​	软连接，源目录/文件是实际存在的，目的目录/文件就相当于“快捷方式”

  ​	如果删除软连接，只需要删除“快捷方式”

- fuser 显示当前哪个进程正在使用磁盘的【某个文件】或【挂载点】，一般配合umount和rm用来解决资源被占用而无法删除的问题

   	fuser -m -v 文件/目录，查看哪个进程占用文件/目录

  ​	fuser -k 文件/目录，杀掉访问文件/目录的进程

  ​	umount 文件/目录，卸载文件/目录

   	rm -rf 文件/目录，删除文件/目录

### 日期命令：

- date 

  修改日期：date -s "月/日/年"

  修改时间：date -s "时:分:秒"

  修改年月日 时分秒：date -s "时:分:秒 月/日/年"

  

### 文件和目录命令:

- cd /home 进入 '/ home' 目录' 

- cd .. 返回上一级目录 

- cd ../.. 返回上两级目录 

- cd 进入个人的主目录 

- cd ~user1 进入个人的主目录 

- cd - 返回上次所在的目录 

- pwd 显示工作路径 

- ls 查看目录中的文件 

- ls -F 查看目录中的文件 

- ls -R 递归显示目录中的文件 

- ls -l 显示文件和目录的详细资料 -h显示可读的文件大小(可自动转化为Mb或者Gb)

- ls -a 显示隐藏文件 

- ls *[0-9]* 显示包含数字的文件名和目录名 

- tree 显示文件和目录由根目录开始的树形结构(1) 

- lstree 显示文件和目录由根目录开始的树形结构(2) 

- mkdir dir1 创建一个叫做 'dir1' 的目录' 

- mkdir dir1 dir2 同时创建两个目录 

- mkdir -p /tmp/dir1/dir2 创建一个目录树 

- rm -f file1 删除一个叫做 'file1' 的文件' 

  ```
  rm -f `ls ./ | grep "abc*"`,删除当前目录下以abc打头的所有文件
  ```

- rmdir dir1 删除一个叫做 'dir1' 的目录' 

- rm -rf dir1 删除一个叫做 'dir1' 的目录并同时删除其内容 

- rm -rf dir1 dir2 同时删除两个目录及它们的内容 

- mv dir1 new_dir 重命名/移动 一个目录 

- cp file1 file2 复制一个文件 

- cp dir/* . 复制一个目录下的所有文件到当前工作目录 

- cp -a /tmp/dir1 . 复制一个目录到当前工作目录 

- cp -a dir1 dir2 复制一个目录  

- tar -zxvf tar文件 -C 指定目录 解压tar压缩包到指定目录

- echo 

  ​		覆盖:	"任意字符" > 某个文本文件  

  ​		追加，保留原来的内容："任意字符" >>  某个文本文件  

- du -b 指定文件 查看指定文件的字节大小

- df -h 查看系统磁盘占用情况

- free -h 查看内存使用情况

- stat 查看文件的详细信息（包括inode和块信息）

- cat 文件名，查看文件的内容

  ​	将文件1和文件2内容合并到文件3，cat 文件1 文件2 > 文件3
  
- zip

  -  -u 【zip压缩包】【要添加的文件1】【文件2】，将一个或多个文件添加到zip压缩包
  - -r  【zip压缩包】【目录】，将目录压缩成zip

### 用户

- 新增用户 useradd 用户名
- 新增密码 passwd 用户名
- 修改用户权限
  -  usermod -g root alex，将普通用户alex添加到root组
- 查看所有用户 users
- 删除用户 userdel 用户名

### 权限：

- chown [-R] 新的用户:新的组 文件/目录，将文件/目录的拥有者指定给新的用户（-R更改对目录下所有文件生效）
- chgrp [-R] 新的组 文件/目录，将文件/目录指定给新的组
- chmod [-R] 权限组合 文件/目录，其中权限组合r-2 w-4 x-1，chmod 777 文件A，文件A提供读/写/执行的所有权限，第一个wrx（所有者权限），第二个wrx（同组的其它用户权限），第三个wrx（其它组的用户权限）

### vi常见命令:

- 复制当前行

  ​	①把光标移动到要复制的行

  ​	②按yy

  ​	③把光标移动到要复制到的位置

  ​	④按p

- 复制多行

  ​	①把光标移动到要复制的行

  ​	②按nyy，n表示从光标处往下要复制的行数

  ​	③把光标移动到要复制到的位置

  ​	④按p

- 删除一行 

  ​	①把光标移动到要删除的行

  ​	②按dd
  
- 删除多行

  ​	①把光标移动到要删除的行

  ​	②按ndd，n表示从光标处往下要删除的行数

- 全选并复制 ggvG

- 移动到首行 gg

- 移动到尾行 G

- 全部删除 esc-dG

- 显示行号 :set nu

- 根据行号删除指定行数据 :开始行号 结束行号d

  - 删除10-100行的数据，:10 100d
  
- 光标移动到行首

  ​	非编辑模式下，按^

- 光标移动到行尾

  ​	非编辑模式下，按$

### linux与window目录共享

```
将window的目录挂载到linux文件系统的根目录下（可以自定义目录，也可以使用/mnt目录）
步骤：
	①设置window下的文件夹为共享文件夹
		选中文件夹 -> 共享 -> 管理员或者everyone -> 共享（访问地址为//ip/文件夹名称）
	②将window的共享文件夹挂载到linux /mnt目录
		mount -t cifs -o username=window系统登录账号,password=密码 //window主机的ip/文件夹名称 /mnt
```

### 常用的远程拷贝命令:

​	①scp，实现服务器与服务器之间的数据安全拷贝

​		基本语法：

​			scp -r 要拷贝的文件路径/名称	目的用户@主机:目的路径/名称	

​	②rsync，远程同步工具(需要yum安装)，用于将文件远程同步到其他服务器的特定目录，比scp速度快，只对差异文件做更新

​		基本语法：

​			rsync 选项 要拷贝的文件路径/名称	目的用户@主机:目的路径/名称

| 选项 | 功能         |
| ---- | ------------ |
| -r   | 递归         |
| -v   | 显示赋值过程 |
| -l   | 拷贝符号连接 |

​	③xsync，通过rsync封装的一个自定义循环

```
#!/bin/sh
#作用：将当前主机的内容远程同步到hadoop03和hadoop04

# 获取输入参数个数，如果没有参数，直接退出
pcount=$#
if((pcount==0)); then
echo no args...;
exit;
fi

# 获取文件名称
p1=$1
fname=`basename $p1`
echo fname=$fname
# 获取上级目录到绝对路径
pdir=`cd -P $(dirname $p1); pwd`
echo pdir=$pdir
# 获取当前用户名称
user=`whoami`
# 循环，将当前服务器指定目录下的文件同步到主机03/04
for((host=3; host<=4; host++)); do
echo $pdir/$fname $user@slave$host:$pdir
echo ==================slave$host==================
rsync -rvl $pdir/$fname $user@hadoop0$host:$pdir
done
#Note:这里的@hadoop0$host是拼接主机名，需要做相应修改。另外，for循环中的host的边界值由自己的主机编号决定。
```

### linux编译的常用命令

```
1、configure

这一步一般用来生成 Makefile，为下一步的编译做准备，你可以通过在 configure 后加上参数来对安装进行控制，比如代码:./configure --prefix=/usr上面的意思是将该软件安装在 /usr 下面，执行文件就会安装在 /usr/bin.同时一些软件的配置文件你可以通过指定 --sys-config= 参数进行设定。有一些软件还可以加上 --with、--enable、--without、--disable 等等参数对编译加以控制，你可以通过允许 ./configure --help 察看详细的说明帮助。

2、make

这一步就是编译，大多数的源代码包都经过这一步进行编译（当然有些perl或Python编写的软件需要调用perl或python来进行编译）。如果 在 make 过程中出现 error ，你就要记下错误代码（注意不仅仅是最后一行），然后你可以向开发者提交 bugreport（一般在 INSTALL 里有提交地址），或者你的系统少了一些依赖库等，这些需要自己仔细研究错误代码。make 的作用是开始进行源代码编译，以及一些功能的提供，这些功能由他的 Makefile 设置文件提供相关的功能，比如 make install 一般表示进行安装，make uninstall 是卸载，不加参数就是默认的进行源代码编译。
make 是 Linux 开发套件里面自动化编译的一个控制程序，他通过借助 Makefile 里面编写的编译规范进行自动化的调用 gcc 、ld 以及运行某些需要的程序进行编译的程序。一般情况下，他所使用的 Makefile 控制代码，由 configure 这个设置脚本根据给定的参数和系统环境生成。

3、make install

这条命令来进行安装（当然有些软件需要先运行 make check 或 make test来进行一些测试），这一步一般需要你有 root 权限（因为要向系统写入文件）
```

### rpm命令

```
在redhat的linux版本可以直接使用rpm命令：
查询是否安装rpm包：
	rpm -qa|grep -i 关键字
安装rpm包：
	rpm -ivh .rpm包
卸载rpm包：
	rpm -e .rpm包
```

