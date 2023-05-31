## 说明
把linux作为一个中转服务器(局域中转，通过中转设备访问linux网络网段下的其他设备)

## 开启路由转发
```shell
#写入配置文件
echo 'net.ipv4.ip_forward=1' > /etc/sysctl.conf

#强制刷新配置文件
sysctl -p
```

## 网络地址转换
```shell
#添加SNAT规则:

sudo iptables -t nat -A POSTROUTING -s <源IP/子网> -o <出口网卡> -j SNAT --to-source <转换后的源IP>

#其中，源IP/子网是指需要修改的源IP地址或者IP地址段，出口网卡是指数据包从哪个网卡出去，转换后的源IP是指将数据包的源IP地址替换为哪个IP地址。
```
```shell
#添加DNAT规则(这个不需要，这里只是做一个参考)：

sudo iptables -t nat -A PREROUTING -d <目标IP> -i <入口网卡> -p <协议> --dport <目标端口> -j DNAT --to-destination <转换后的目标IP:端口号>

#其中，目标IP是指需要修改的目标IP地址，入口网卡是指数据包从哪个网卡进来，协议是指需要转换的协议类型，目标端口是指需要转换的目标端口，转换后的目标IP是指将数据包的目标IP地址替换为哪个IP地址和端口号。
```
```shell
#保存规则
sudo iptables-save > /etc/iptables/rules.v4
```
```shell
#允许数据包转发
echo 1 >/proc/sys/net/ipv4/ip_forward
sysctl -w net.ipv4.conf.eth0.route_localnet=1
sysctl -w net.ipv4.conf.default.route_localnet=1
```


```
总结：

1，NAT功能主要是玩的iptables的NAT表。

2，代理内网机器上网，需要设置的是POSTROUTING链

3，转发外网端口流量则是设置PREROUTING链
```

## 参考链接
[内网穿透工具ZeroTier...](https://www.bilibili.com/video/BV1Vh411F7Mr/?share_source=copy_web&vd_source=988ba137d6a6f8fc134601e37b976fba)

[iptables 网络地址转换NAT](https://blog.csdn.net/jrunw/article/details/95332258)

[如何使用iptables和NAT](https://blog.csdn.net/weixin_43402206/article/details/126529113)
