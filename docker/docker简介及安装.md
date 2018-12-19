#### Docker的应用场景

- Web 应用的自动化打包和发布。
- 自动化测试和持续集成、发布。
- 在服务型环境中部署和调整数据库或其他的后台应用。
- 从头编译或者扩展现有的OpenShift或Cloud Foundry平台来搭建自己的PaaS环境。

#### Docker 的优点

- 简化程序
- 避免选择恐惧症
- 节省开支

#### Centos Docker安装

1. 需要检查linux的版本是否支持

Docker 要求 CentOS 系统的内核版本高于 3.10 ，查看本页面的前提条件来验证你的CentOS 版本是否支持 Docker 。

~~~sh
uname -r
3.10.0-514.el7.x86_64
~~~

2. 安装一些必要的系统工具

~~~sh
yum install -y yum-utils device-mapper-persistent-data lvm2
~~~

3. 添加软件源信息 

~~~sh
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
~~~

4. 更新 yum 缓存

~~~sh
sudo yum makecache fast
~~~

5. 安装 Docker-ce：

```
sudo yum -y install docker-ce
```

6. 启动 Docker 后台服务

```
sudo systemctl start docker
```

7. 测试运行 hello-world

```
[root@runoob ~]# docker run hello-world
```