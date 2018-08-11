### GitHub Clone慢Push慢Pull慢，几十kb/s的clone速度

在hosts文件添加 两个地址（Linux修改/etc/hosts|windows下C:\Windows\System32\drivers\etc\hosts)

```
151.101.113.194 github.global.ssl.fastly.net
192.30.253.113 github.com
```

#### 第一个地址怎么确定?

> 点击这个链接 [查询](http://github.global.ssl.fastly.net.ipaddress.com/)
>
> 输入要查询的域名 : github.global.ssl.fastly.net

#### 第二个地址怎么确定?

> 点击这个链接 [查询](http://tool.chinaz.com/dns/)
>
> 输入要查询的域名 : github.com

