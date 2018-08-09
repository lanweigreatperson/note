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

