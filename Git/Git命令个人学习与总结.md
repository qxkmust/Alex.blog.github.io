# Git命令个人学习与总结

### *一般推荐安装Git后,在git-bash中输入git命令:*



- ##### 初始化本地仓库

  ```
  git init
  ```

  

- ##### 切换到要提交的分支

```
git checkout 分支名
```



- ##### 关联本地与远程分支

```
git pull origin 分支名 --allow-unrelated-histories
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

  
