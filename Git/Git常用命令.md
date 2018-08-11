**[Git](https://git-scm.com/docs)官网文档**

**Git安装**

- [Mac](https://sourceforge.net/projects/git-osx-installer/files/git-2.18.0-intel-universal-mavericks.dmg/download?use_mirror=autoselect)
- [Windows](https://github.com/git-for-windows/git/releases/download/v2.18.0.windows.1/Git-2.18.0-64-bit.exe)

安装完成后，还需要最后一步设置，在命令行输入： 

~~~
$ git config --global user.name "Your Name"
$ git config --global user.email "email@example.com"
~~~

**创建版本库**

```java
--创建文件夹
mkdir learngit
--切换到文件夹下
cd learngit
--把这个目录变成Git可以管理的仓库
git init
--接下来可以创建文件了
vi readme.txt
--git add告诉Git，把文件添加到仓库， 提交多个文件使用空格隔开
git add readme.txt
--git commit告诉Git，把文件提交到仓库
git commit -m "wrote a readme file"
```

**时光穿梭**

~~~
--命令可以让我们时刻掌握仓库当前的状态
git status
--具体修改了什么内容
git diff
~~~

**版本回退**

~~~
--查看提交历史
git log --pretty=oneline
--回退到上一个版本 当然往上100个版本写100个^比较容易数不过来，所以写成HEAD~100
git reset --hard HEAD^
--切换到某个版本
git reset --hard 1094a
--查看命令历史
git reflog

~~~

**工作区和暂存区**

**管理修改**

用`git diff HEAD -- readme.txt`命令可以查看工作区和版本库里面最新版本的区别 

**撤销修改**

`git checkout -- file`可以丢弃工作区的修改 ,`git checkout -- file`命令中的`--`很重要，没有`--`，就变成了“切换到另一个分支”的命令 

`git reset HEAD <file>`可以把暂存区的修改撤销掉 

`git reset`命令既可以回退版本，也可以把暂存区的修改回退到工作区。当我们用`HEAD`时，表示最新的版本 

**删除文件**

一种是从版本库中删除该文件，那就用命令`git rm`删掉 ,并且`git commit` 

另一种情况是删错了，因为版本库里还有呢，所以可以很轻松地把误删的文件恢复到最新版本：

`git checkout -- test.txt`

**远程仓库**

- 创建SSH Key
  - 在用户主目录下，看看有没有.ssh目录，如果有，再看看这个目录下有没有`id_rsa`和`id_rsa.pub`这两个文件，如果已经有了，可直接跳到下一步 ，如果没有创建`ssh-keygen -t rsa -C "youremail@example.com"`注意换成自己的邮箱
  - 登陆GitHub，打开“Account settings”，“SSH Keys”页面，点“Add SSH Key”，填上任意Title，在Key文本框里粘贴`id_rsa.pub`文件的内容 

**添加远程库** 

- 在github上创建仓库，创建成功后根据提示将本地仓库推送至github上，也可以从github上克隆出新仓库

  ### …or create a new repository on the command line

  ```
  echo "# note" >> README.md
  git init
  git add README.md
  git commit -m "first commit"
  git remote add origin https://github.com/lanweigreatperson/note.git
  git push -u origin master
  ```

  ### …or push an existing repository from the command line

  ```
  git remote add origin https://github.com/lanweigreatperson/note.git
  git push -u origin master
  ```

从现在起，只要本地作了提交，就可以通过命令：

```
$ git push origin master
```

总结：要关联一个远程库，使用命令`git remote add origin git@server-name:path/repo-name.git`；

关联后，使用命令`git push -u origin master`第一次推送master分支的所有内容；

此后，每次本地提交后，只要有必要，就可以使用命令`git push origin master`推送最新修改；

分布式版本系统的最大好处之一是在本地工作完全不需要考虑远程库的存在，也就是有没有联网都可以正常工作，而SVN在没有联网的时候是拒绝干活的！当有网络的时候，再把本地提交推送一下就完成了同步，真是太方便了！

**从远程库克隆**

```
git clone git@github.com:michaelliao/gitskills.git
```

**分支管理**

**创建与合并分支**

```
--创建dev分支，然后切换到dev分支
$ git checkout -b dev
Switched to a new branch 'dev'
--git checkout命令加上-b参数表示创建并切换，相当于以下两条命令
$ git branch dev
$ git checkout dev
Switched to branch 'dev'
--git branch命令查看当前分支
$ git branch
* dev
  master
--接下来就可以对分支操作了
....git add ***、git commit -m....
--切换回master分支
$ git checkout master
Switched to branch 'master'
--git merge命令用于合并指定分支到当前分支
$ git merge dev
Updating d46f35e..b17d20e
Fast-forward
 readme.txt | 1 +
 1 file changed, 1 insertion(+)
 --删除dev分支
 $ git branch -d dev
Deleted branch dev (was b17d20e).
--删除后，查看branch，就只剩下master分支
$ git branch
* master
```