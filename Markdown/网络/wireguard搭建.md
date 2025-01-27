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

```c++
使用Remove-NetNat可以清空Nat网络
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

# iperf测试丢包率
```shell
#服务端
iperf3 -s
```

```shell
#客户端
./iperf3.exe -R -c 10.100.1.1 -u -b 1024K -t 10

# -R 反向模式运行（服务器发送，客户端接收）
# -c 服务端ip
# -u 使用UDP
# -b 设置带宽速度
# -t 测试时间10ms

```

#### 定义表格

|时间间隔|数据传输量|比特率|抖动|丢失/总共的报文|
|----|----|----|----|----|
|0.00-1.00|817 KBytes|6.69 Mbits/sec| 0.029 ms|丢失/总共的报文 
|||||接收方（sender）
|||||发送方（receiver）

