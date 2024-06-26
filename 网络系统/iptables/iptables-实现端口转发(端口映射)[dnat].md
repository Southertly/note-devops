# iptables实现端口转发(端口映射)

环境描述

A: 作为转发服务器, 作为相当于路由, 开放端口8080

B: 后端实际服务器, 开放端口80

客户端通过访问A的8080端口, 获取B的80端口的响应.

以下操作都是在A中执行

## 开启Linux的数据转发

```log
$ sysctl -a | grep ip_forward
net.ipv4.ip_forward = 0
```

如果`net.ipv4.ip_forward`的值为1, 则不必修改. 如果为零, 则需要执行如下操作

```
echo 'net.ipv4.ip_forward=1' >> /etc/sysctl.conf
sysctl -p
```

## 设置iptables的端口映射

将来自客户端的, 目标是A服务器的8080端口的请求, 重写为访问到B服务器的80端口的请求

```
iptables -t nat -A PREROUTING -p tcp -m tcp --dst A的IP地址 --dport 8080 -j DNAT --to B的IP地址:80
```

将由A服务器转发出去的目标是B服务器80端口的请求, 添加MASQUERADE标记

```
iptables -t nat -A POSTROUTING -p tcp -m tcp --dst B的IP地址 --dport 80 -j MASQUERADE
```

然后要确认B服务器上的80端口上确实有ACCEPT规则

- -p protocol
- -m match, 匹配
- -d/--dst 请求的目标地址
- -s/--src 请求的来源地址(POSTROUTING链上的规则添加`-s`选项可以用来伪装IP)
- --dport/--sport 请求的目标/来源端口

建议保存设置并重启服务, 保存设置执行`service iptables save`命令即可.

为所有来源添加`MASQUERADE`标记

```
iptables -t nat -A POSTROUTING -s 0/0 -j MASQUERADE
```
