gitlab的安装参考：https://blog.csdn.net/ouyang_peng/article/details/72903221



gitlab 常用的几个命令

~~~
# 重新应用gitlab的配置,每次修改/etc/gitlab/gitlab.rb文件之后执行
sudo gitlab-ctl reconfigure

# 启动gitlab服务
sudo gitlab-ctl start

# 重启gitlab服务
sudo gitlab-ctl restart

# 查看gitlab运行状态
sudo gitlab-ctl status

#停止gitlab服务
sudo gitlab-ctl stop

# 查看gitlab运行所有日志
sudo gitlab-ctl tail

#查看 nginx 访问日志
sudo gitlab-ctl tail nginx/gitlab_acces.log 

#查看 postgresql 日志
sudo gitlab-ctl tail postgresql 

# 停止相关数据连接服务
gitlab-ctl stop unicorn
gitlab-ctl stop sidekiq

# 系统信息监测
gitlab-rake gitlab:env:info  

~~~



root密码忘记解决办法

~~~sh
gitlab-rails console production
-------------------------------------------------------------------------------------
 GitLab:       11.1.4 (63daf37)
 GitLab Shell: 7.1.4
 postgresql:   9.6.8
-------------------------------------------------------------------------------------
Loading production environment (Rails 4.2.10)
irb(main):001:0> user = User.where(id: 1).first
=> #<User id:1 @root>
irb(main):002:0> user.password=12345678
=> 123456
irb(main):003:0> user.password_confirmation=12345678
=> 123456
irb(main):004:0> user.save!
=> true
irb(main):008:0> quit
~~~



