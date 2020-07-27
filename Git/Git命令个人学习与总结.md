# Git命令个人学习与总结

### *一般推荐安装Git后,在git-bash中使用git命令:*

- 关联github

  ```
  登录github：
  git config --global user.email "1841359714@qq.com"
  git config --global user.name "qxkmust"
  ```

- ##### 初始化本地仓库

  ```
  git init
  注意：可以解决将.git误删，重新生成.git目录
  ```

- 拉取远程分支

```
git pull origin 分支名 --allow-unrelated-histories
```

- 创建分支

  ```
  git branch 分支名
  注意：要commit之后才能创建分支，否则报错“Not a valid object name:master”
  ```

- ##### 切换到要提交的分支

```
git checkout 分支名
```

- ##### 首先进行git add 添加文件

```
git add 当前目录某个文件名

*或者*

当前目录所有(已修改)文件 git add .

warning: LF will be replaced by CRLF
最简单的一种办法就是把自动转换功能关掉即可。
输入命令 ：git config core.autocrlf false (仅对当前git仓库有效）
git config --global core.autocrlf false (全局有效）
```



- #### git commit 提交

```
使用git commit 的时候会有几个参数，git commit -m 打出你要提交的日志说明，如果么有的话，会进入一个vb编辑页面，简单的话，还是直接-m打出文字，直接enter即可。
```



- #### commit之后push

```
请注意，这个时候你只是把文件提交到了自己本地，并没有提交到服务器端，这个时候就要用到git push
 命令行也行，或者是直接win推送就可以了，这个时候即使win提交Git修改文件卡死，也能愉快的修改提交代码了。

可能遇到以下情况,提交不成功:

 **使用git push origin master报错:“fatal: ‘origin’ does not appear to be a git repository
fatal: Could not read from remote repository.** 
 原因：没有关注远程仓库为origin
 解决：git remote add origin “远程仓库地址” 

 **使用git push origin master报错:
hint: Updates were rejected because the tip of your current branch is behind
hint: its remote counterpart. Integrate the remote changes (e.g.
hint: 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.**
 原因: 远程repository和我本地的repository冲突造成
 解决1: 通过新建分支git branch 新的分支名;再次push 到新的分支名
 
 解决2：git push -f [remote url]
 如果不想再新建分支，可以强行将本地版本推送到远程/分支，但是可能将另一个人的改动覆盖掉，不建议使用
 
 解决3：先 git fetch [remote url] 然后git merge [remote url]
 获取远程分支的变动，与本地分支合并, 之后再push。（但是不那么奏效，第二种虽然暴力，但是有效）
 
 解决4：先 git ls-files查看已经放入commit缓存的文件，然后使用git rm --cached -r "window下的文件目录"，先将commit缓存删除后重新commit

如果以上办法都不可行:
1. 直接切换到提交的分支 git checkout 分支名
2. 提交到远程仓库地址 git push 远程仓库地址 分支名
```

------

- 将远程最新版本更新到本地

  ```
  1、查看远程仓库 git remote -v
      origin  https://github.com/qxkmust/Alex.blog.github.io.git (fetch)
      origin  https://github.com/qxkmust/Alex.blog.github.io.git (push)
     其中，origin（指代远程仓库）
  2、从远程获取最新版本到本地 git fetch origin 远程分支:临时分支temp
  3、比较本地仓库与下载的temp分支 git diff temp,按q“跳出”
  4、合并temp分支到本地版本 git merge temp  （如果无法合并，需要使用git rebase）
  5、删除临时分支 git branch -d temp
  ```


- git拉取远程分支的两种方式

  ```
  1、当本地有其他分支的代码仓库时
  
  通过下述命令查看所有的远程分支：
  git branch -r
  下面有2种方法来拉取远程分支代码：
  (1).需要本地分支和远程分支建立映射关系
  执行如下命令：
  git checkout -b 本地分支xxx  origin/远程分支xxx
  使用这种方式会在本地仓库新建本地分支xxx，并自动切换到新建的本地分支xxx，当然了远程分支xxx的代码也拉取到了本地分支xxx中。采用这种方法建立的本地分支会和远程分支建立映射关系。
  (2).不需要本地分支和远程分支建立映射关系
  执行如下命令：
  git fetch origin 远程分支xxx:本地分支xxx
  使用这种方式会在本地仓库新建本地分支xxx，但是并不会自动切换到新建的本地分支xxx，需要手动checkout，当然了远程分支xxx的代码也拉取到了本地分支xxx中。采用这种方法建立的本地分支不会和远程分支建立映射关系。
  
  2、当本地没有其他分支的代码仓库时
  
  执行如下命令：
  git clone -b 分支名 仓库地址
  执行上述命令后就将远程分支拉取到了本地。
  ```

## 常见问题

```
问题1：
Failed to connect to github.com port 443: Timed out？
分析：可能是在hosts配置了域名映射失效了
解决办法：如下
```

![1595863069109](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1595863069109.png)

```
问题2：
remote: Invalid username or password.
fatal: Authentication failed for 'https://github.com/qxkmust/Alex.blog.github.io.git/'
分析：可能更改了github的认证信息，需要在git中重新认证
解决：
	在git bash中使用命令：
	git config --global user.email "github邮箱"
	git config --global user.name "github用户名"
```

