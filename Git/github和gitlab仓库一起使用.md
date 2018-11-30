### `Github`和`Gitlab`仓库一起使用

作为开发人员，在同一台电脑上一般都会同时用两者，一个是作为私人，一个是为公司服务。

如何在一台机器上面同时使用 `Github` 与 `Gitlab` 的服务.参考网上的操作。

#### 生成秘钥

公司`GitLab`生成一个SSH-Key

~~~sh
ssh-keygen -t rsa -C "注册的gitlab邮箱" -f ~/.ssh/gitlab_id-rsa
~~~

公网`Github`生成一个SSH-Key

~~~sh
ssh-keygen -t rsa -C "注册的github邮箱" -f ~/.ssh/github_id-rsa
~~~

会在用户目录下的`.ssh`目录下生成相应的文件。

把内容分别复制到`Gitlab`和`Github`

####  添加config

在`~/.ssh`下添加config配置文件,内容如下：

~~~
# github key
Host github
    Port 22
    User git
    HostName github.com
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/github_id-rsa
Host gitlab
    Port 22
    User git
    HostName gitlab.com
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/gitlab_id-rsa
~~~

下面对上述配置文件中使用到的配置字段信息进行简单解释：

~~~
Host
    它涵盖了下面一个段的配置，我们可以通过他来替代将要连接的服务器地址。
    这里可以使用任意字段或通配符。
    当ssh的时候如果服务器地址能匹配上这里Host指定的值，则Host下面指定的HostName将被作为最终的服务器地址使用，并且将使用该Host字段下面配置的所有自定义配置来覆盖默认的`/etc/ssh/ssh_config`配置信息。
Port
    自定义的端口。默认为22，可不配置
User
    自定义的用户名，默认为git，可不配置
HostName
    真正连接的服务器地址
PreferredAuthentications
    指定优先使用哪种方式验证，支持密码和秘钥验证方式
IdentityFile
    指定本次连接使用的密钥文件
~~~

#### 配置仓库

假设`Gitlab`与`Github`的工作目录分别如下所示：

```
github工作仓库:~/workspace/github
gitlab工作仓库:~/workspace/gitlab
```

则配置如下：

```
#gitlab
cd ~/workspace/gitlab
git init
git config --global user.name 'gitlab'
git config --global user.email 'gitlab@company.com'

#github
cd ~/workspace/github
git init
git config --local user.name 'personal'
git config --local user.email 'personal@163.com'
```

#### 测试

```
# 测试github
$ ssh -T git@github.com
 
# 测试gitlab
$ ssh -T git@gitlab.com
```