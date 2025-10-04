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

sudo iptables -t nat -A POSTROUTING -s <IP/子网> -o <出口网卡> -j SNAT --to-source <转换后的IP>

#其中，IP/子网是指需要修改的IP地址或者IP地址段，出口网卡是指数据包从哪个网卡出去。
#比如zerotier要通过服务器的网络访问，IP/子网就是zerotier网段，出口网卡是服务器的网卡，转换回的IP是服务器IP
```

```shell
#添加DNAT规则(这个不需要，这里只是做一个参考)：

sudo iptables -t nat -A PREROUTING -d <IP> -i <入口网卡> -p <协议> --dport <端口> -j DNAT --to-destination <转换后的IP:端口号>

#其中，IP是指需要修改的IP地址，入口网卡是指数据包从哪个网卡进来，协议是指需要转换的协议类型。
#--dports 10000:50000
#外网访问通过服务器转发到zerotier设备，IP是外网网段，入口网卡服务器一般是eth0，转换后的IP是zerotier设备这个可以是家庭设备。（这样当访问公网服务器会通过VPN转发到指定设备，从而达到反向代理）
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

```shell
#查看nat规则
iptables -t nat -L

#删除nat规则
iptables -t nat -D POSTROUTING <规则号>
```

    总结：

    1，NAT功能主要是玩的iptables的NAT表。

    2，代理内网机器上网，需要设置的是POSTROUTING链

    3，转发外网端口流量则是设置PREROUTING链


## 下发IPV6地址

### 安装 ndppd
框架已经搭建好了，现在任何一台通过该 zerotier 网络认证的客户端都会获得一个可用的公网 IPv6 地址，它们彼此之间可以通信。但是现在为止除 ECS 服务器外，其他客户端依然无法访问外部 IPv6 地址及被其他 IPv6 地址访问。这是因为zerotier 申明的公网 IPv6 地址未进行正常的 IPv6 广播，不能被其他的 IPv6 地址知道。

在 ECS 服务器上安装 ndppd 软件以支持 IPv6 地址广播，安装后的 `/etc/ndppd.conf`配置文件内容如下。eth0 修改为 ECS 服务器的默认网卡，rule 修改为对应 IPv6 地址网段。

```bash
# 开启转发
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
echo "net.ipv4.conf.all.forwarding = 1" >> /etc/sysctl.conf
echo "net.ipv6.conf.default.forwarding = 1" >> /etc/sysctl.conf
echo "net.ipv6.conf.all.forwarding = 1" >> /etc/sysctl.conf
echo "net.ipv6.conf.all.proxy_ndp = 1" >> /etc/sysctl.conf
echo "net.ipv6.conf.all.accept_ra = 2" >> /etc/sysctl.conf
```

```bash
# 开启 zerotier 启用默认路由
# 每个需要IPV6公网都需要配置
sudo zerotier-cli set b15644912e8cf918 allowGlobal=true
sudo zerotier-cli set b15644912e8cf918 allowDefault=1

# zerotier开启IPV6
# 两个选项默认打开，无需修改任何
# 记住ZeroTier RFC4193这个后续路由需要用到；这个 ::/0（::/0 via	fdb1:5644:912e:8cf9:1899:93a5:b0fa:ad22）
# 开启 Auto-Assign from Range 这个IP段根据自己的公网IP配置
# 不要忘记添加IPV6的默认路由（240e:33d:9140:f00::/64    (LAN)   ）
```

```bash
# 安装 ndppd
sudo apt install -y ndppd
```

```conf
# 配置文件；注意填写自己的公网IPV6
# 注：IPV6地址大小写
route-ttl 30000

address-ttl 30000

proxy eth0 {
    router yes
    timeout 500
    autowire no
    keepalive yes
    retries 3
    ttl 30000
    rule 2001:470:811d::/48 {
        auto
        autovia no
    }
}
```

```bash
# 重启 ndppd
sudo systemctl restart ndppd
```


## 参考链接

[内网穿透工具ZeroTier...](https://www.bilibili.com/video/BV1Vh411F7Mr/?share_source=copy_web\&vd_source=988ba137d6a6f8fc134601e37b976fba)

[iptables 网络地址转换NAT](https://blog.csdn.net/jrunw/article/details/95332258)

[如何使用iptables和NAT](https://blog.csdn.net/weixin_43402206/article/details/126529113)



## 搭建 Zerotier Moon 为虚拟网络加速

```bash
# 进入 zerotier-one 程序所在的目录，默认为 `/var/lib/zerotier-one`。
cd /var/lib/zerotier-one

# 生成 moon.json 配置文件
zerotier-idtool initmoon identity.public >> moon.json

# 编辑 moon.json 配置文件
vim moon.json
```

将配置文件中的 `"stableEndpoints": []` 修改成 `"stableEndpoints": ["ServerIP/9993"]`，将 ServerIP 替换成云服务器的公网IP。

```bash
# 生成 .moon 文件
zerotier-idtool genmoon moon.json

# 将生成的 000000xxxxxxxxxx.moon 移动到 moons.d 目录
mkdir moons.d
mv 000000xxxxxxxxxx.moon moons.d

# 重启 zerotier-one 服务
systemctl restart zerotier-one
```


### 使用 Moon
普通的 Zerotier 成员使用 Moon 有两种方法，第一种方法是使用 `zerotier-cli orbit` 命令直接添加 Moon 节点ID；第二种方法是在 `zerotier-one` 程序的根目录创建`moons.d`文件夹，将 `xxx.moon` 复制到该文件夹中，我们采用第一种方法

```bash
# 例如
zerotier-cli orbit xxxxxxxxxx xxxxxxxxxx

# 查看是否加入moon；输出列表有moon代表连接moon节点否则没有使用moon
zerotier-cli listpeers


#根目录
Linux：/var/lib/zerotier-one
Windows：C:\ProgramData\ZeroTier\One

```