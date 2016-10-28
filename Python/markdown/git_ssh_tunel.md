##git远程外网地址变内网怎么破(ssh本地端口转发)？

最近给longtubas上了负载均衡，相对来说我们并发并不高，但希望可用性尽可能高，本来打算用不饱和的机器做个lvs集群的，

但IDC说给个额外的公网ip需要申请机器，推荐我们用云自带的负载均衡，倒也省的自己配置，IDC把之前公网ip绑定到负载均衡器上了，

这导致我们搭建在服务器上的gitlab不可用了，折腾了一下，使用ssh本地端口转发解决了。

当然前提是你有另外一台机器能通过内网连接到web服务器

A: 192.168.3.42(内网) web服务器

B: 192.168.3.29(内网) 42.62.14.7(外网) 中转服务器

C: 本机(只有内网地址)


##修改git remote url

我们之前是用git协议访问的，修改为ssh访问

在C上修改.git/config
```
[remote "origin"]
    url = ssh://git@42.62.14.7:2222/bi/python.git
```

##ssh本地端口转发

在B上做端口转发,将来自端口2222的数据转发到192.168.3.42 22号端口
```
ssh -NfL 0.0.0.0:2222:192.168.3.42:22 root@192.168.3.42
```

恩，这样就可以了

在C上面还可以通过3.42的2222端口直接ssh到A主机

ssh root@42.62.14.7 -p 2222

##ssh还可以做远程端口转发

当然ssh还可以做远程端口转发，假如我希望通过B主机能ssh到我C的机器

在C主机上面
```
ssh -CNfR 9999:127.0.0.1:22 root@42.62.14.7
```

在B主机上面可通过9999端口ssh到C主机
```
ssh skycrab@localhost -p 9999
```

对于负载均衡的健康检查多说一句，建议使用http方式，而且path的请求最好是通过应用服务器返回的，

而不是通过nginx配置的，这样应用服务器挂了，status_code不是200也会被摘除的。

当然也可以使用nginx做健康检查，尤其是淘宝tengine自带的健康检查模块[http_upstream_check](http://tengine.taobao.org/document_cn/http_upstream_check_cn.html)


