# 安装
```shell
apt install wireguard

# 加载内核
modprobe wireguard

# 查看内核加载情况
lsmod | grep wireguard

# 查看tun
cat /dev/net/tun
# cat: /dev/net/tun: File descriptor in bad state（返回正常）
```

# 配置
```shell
# 生成密钥

wg genkey | sudo tee /etc/wireguard/privatekey-wg0 | wg pubkey | sudo tee /etc/wireguard/publickey-wg0
```
```shell
# 配置文件

vim /etc/wireguard/wg0.conf
```
## 写入配置信息
```
[Interface]
Address = 10.100.1.1/32, fd23:23:23::1/128
SaveConfig = true
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
ListenPort = 51800
PrivateKey = SERVER_PRIVATE_KEY  #服务器端生成的私钥

[Peer]
PublicKey = Peer #客户端生成的公钥 publickey
AllowedIPs = 10.100.1.2/32, fd23:23:23::2/128
PersistentKeepalive = 15
```

# 启动
```shell
wg-quick up wg0
wg-quick down wg0
```


## Windos服务端
[微软地址'设置 NAT 网络'](https://learn.microsoft.com/zh-cn/virtualization/hyper-v-on-windows/user-guide/setup-nat-network)

```bash
# 查找刚创建的虚拟交换机的接口索引
Get-NetAdapter
```

```C++
// 使用 New-NetIPAddress 配置 NAT 网关。
New-NetIPAddress -IPAddress 10.100.1.1 -PrefixLength 24 -InterfaceIndex 70

/*
IPAddress - NAT 网关 IP 指定要用作 NAT 网关 IP 的 IPv4 或 IPv6 地址。常规形式将为 a.b.c.1（例如 172.16.0.1）。 尽管最后一个位置不一定必须是.1，但通常是（基于前缀长度）
通用网关 IP 为 192.168.0.1

PrefixLength -- NAT 子网前缀长度定义的 NAT 本地子网大小（子网掩码）。 子网前缀长度将介于 0 到 32 之间的一个整数值。
0 将映射整个 Internet，32 则只允许一个映射的 IP。 常用值范围从 24 到 12，具体要取决于多少 IP 需要附加到 NAT。
常用 PrefixLength 为 24 -- 这是子网掩码 255.255.255.0

InterfaceIndex -- ifIndex 是你在上一步中确定的虚拟交换机的接口索引。
*/
```

```C++
// 若要配置网关，你将需要提供一些有关网络和 NAT 网关的信息：
New-NetNat  -Name MyNATnetwork -InternalIPInterfaceAddressPrefix 10.100.1.0/24

/*
Name - NATOutsideName 描述 NAT 网络的名称。 将使用此参数删除 NAT 网络。

InternalIPInterfaceAddressPrefix - NAT 子网前缀同时描述上述 NAT 网关 IP 前缀和上述 NAT 子网前缀长度。

New-NetNat  -Name MyNATnetwork -InternalIPInterfaceAddressPrefix 10.100.1.0/24
*/
```

```
使用Remove-NetNat可以清空Nat网络
```

```shell
# Windows 设置路由
# 要在Windows操作系统上配置路由，以使得WireGuard客户端之间的内网能够互通，可以使用以下步骤：

# 查找到WireGuard VPN接口，并记下其接口索引号。
route print

# route add 目标网络地址 MASK 子网掩码 WireGuard接口索引号
route add 192.168.1.0 mask 255.255.255.0 11

```


# 客户端
## win
```
wg genkey | sudo tee /etc/wireguard/privatekey-wg0 | wg pubkey | sudo tee /etc/wireguard/publickey-wg0 #客户端和服务生成的密钥指令一样

[Interface]
PrivateKey = CLIENT_PRIVATE_KEY  #客户端生成的私钥
Address = 10.100.1.2/32, fd23:23:23::2/128 #改为想给本机分配的ip
DNS = 114.114.114.114   #dns地址

[Peer]
PublicKey = SERVER_PUBLIC_KEY  #服务器端生成的公钥
Endpoint = SERVER_IP_ADDRESS:51800  #服务器的公网IP和端口
AllowedIPs = 10.100.1.0/24, ::/0	#这里是限制只有10.100.1 网段的请求走WireGuard,不影响其它应用上网,设为0.0.0.0/0则是所有请求均走WireGuard
PersistentKeepalive = 120	#握手时间，每隔120s ping一次客户端
```
## Linux
```
//然导入了配置文件，但是直接运行的时候出错了，提示网络错误，后来查阅各方资料发现对于Linux系统，必须要设定本机网络参数。

// 查看路由表信息
ip route list table main default 
ip -brief address show eth0

// 网络启动，关闭执行的指令路由表
// 添加一条规则，将来自 IP 地址 192.168.1.100 的流量通过路由表 200 进行路由。
PostUp = ip rule add table 200 from 192.168.1.100
PostUp = ip route add table 200 default via 192.168.1.1
PreDown = ip rule delete table 200 from 192.168.1.100
PreDown = ip route delete table 200 default via 192.168.1.1
```


# wireguard映射内网端口
+ 注意客户端ip要代理网络请求（如果只有1.1.1.1那么只代理1.1.1.1，不确定ip最好全局）
+ 添加DNAT规则映射端口信息，让服务器的端口映射到内网客户端


# 为客户端分配公网IPV6
ipv4目前还没有研究是否可行

ipv6配置思路
```bash
# 准备工作
服务器
OpenWrt（路由器）


# 搭建思路
#以下ipv6为简要说明，具体配置看情况
服务器IPV6地址为：2a06:de00:503:5df1::1/64
服务端分配ipv6：2a06:de00:503:5df2::/64（注意这里是wireguard配置文件）
客户端OpenWrt分配ip：2a06:de00:503:5df3::3/64（注意这里是wireguard配置文件）

# OpenWrt（路由器）修改设置
OpenWrt修改走代理的ipv6为 ::/0
OpenWrt 添加网口为wireguard的网口，网关填写 2a06:de00:503:5df2:: ;IPV6分配地址写为 2a06:de00:503:5df3::3/64



```


# WireGuard 配置 Global IPv6 （公网下放）
[WireGuard 配置 Global IPv6 （公网下放）](https://www.xiaoxk.com/wireguard-global-ipv6)
众所周知，IPv6 和 IPv4 不同，IPv6 的地址池更加丰富，也摒弃了 IPv4 中目前常见的 NAT 做法（虽然有 NAT6 的存在，但这终究是一种不受推荐的选择）。对于如果 WireGuard 中的一个 Peer 存在 IPv6 前缀（即 IP 地址不是/128），该设备就可以作为 Server，将前缀下的一部分地址分配给 VPN 中的其他设备的。

基本说明
```
1、邻居列表
  邻居列表主要显示无状态ipv6地址。在IPv6环境中，通过邻居或者到达邻居的通信，会因各种原因而中断。因此节点需要维护一张邻居表。在邻居表中，每个邻居都有相应的状态，并且状态之间可以迁移。

【状态】分为以下六种：
未完成(Incomplete)：表示正在解析地址，但邻居链路层地址尚未确定。

可达(Reachable)：表示地址解析成功，30s内有数据传输，该邻居可达。

陈旧(Stale)：表示可达时间耗尽，30s左右没有数据传输，并不代表地址不可用。如果想要从陈旧状态变为可达状态，可以在应用工具--Ping测试中，输入状态显示陈旧的IPv6地址，协议类型选择IPv6，选择接口去ping这个IPv6地址，状态即可变为可达状态。

延迟(Delay)：邻居可达性未知。Delay状态不是一个稳定的状态，而是一个延时等待状态。

探测(Probe)：邻居可达性未知,正在发送邻居请求探针以确认可达性.

EMPTY（空闲状态）：表示节点上没有相关邻接点的邻居缓存表项。
```

修改`/etc/sysctl.conf`或创建`/etc/sysctl.d/wireguard.conf`文件（文件名随意），更改后添加下面参数，启用对应的功能。
```bash
net.ipv4.ip_forward=1        # IPv4 的网络转发设置
net.ipv6.conf.all.forwarding=1        # IPv6 的网络转发设置
# net.ipv6.conf.default.forwarding=1     # IPv6 的网络转发设置（后续新建接口）
net.ipv6.conf.eth0.proxy_ndp=1    # 启用 NDP（Neighbor Discovery Protocol）协议，让主机作为代理响应 NDP 请求。
```
之后通过`sysctl -p`或重启使设置生效。
执行下面的命令，给客户机的 IPv6 地址设置 NDP 代理：
```ip -6 neigh add proxy 2408:****:6666::2 dev eth0```
删除时将`add`参数修改为`del`即可。

执行命令`systemctl restart systemd-networkd`重启网络，使 NDP Proxy 生效。

允许网内流量转发：`ip6tables -A FORWARD -i wg0 -j ACCEPT`
允许来自公网的流量进入：`ip6tables -A FORWARD -i eth0 -o wg0 -j ACCEPT`
如果不生效，可以调整下规则顺序，把 `-A FORWARD`改为 `-I FORWARD 1` 。

## 文件配置
服务端
```
[Interface]
PrivateKey = :) # 记得改成自己的
ListenPort = 8501
Address = 10.100.1.1/32, 2a06:de00:503:5dfe::1/128

[peer]
publickey = :) # 记得改成自己的
allowedips = 10.100.1.2/32, 2a06:de00:503:5dfe::2/128
```
客户端
```
[Interface]
PrivateKey = :)  # 记得改成自己的
Address = 10.100.1.2/32, 2a06:de00:503:5dfe::2/128
DNS = 240c::6666, 223.5.5.5  # 根据需要设置

[Peer]
PublicKey = :)  # 记得改成自己的
AllowedIPs = 10.100.1.0/24, ::/1, 8000::/1
Endpoint = :) # Server 地址， IPv4/IPv6 均可，改成自己的。
PersistentKeepalive = 120
```
注：`::/0`可能存在一个问题，就是它也包括了所有的 IPv4 流量。如果不需要 IPv4 的流量进入 VPN 网络，可拆分成两个网段（`::/1, 8000::/1`）的写法，避免生成高优先级的默认路由。
