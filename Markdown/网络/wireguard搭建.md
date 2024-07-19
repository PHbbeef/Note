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
Address = 10.100.1.1/24
SaveConfig = true
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
ListenPort = 51800
PrivateKey = SERVER_PRIVATE_KEY  #服务器端生成的私钥

[Peer]
PublicKey = Peer #客户端生成的公钥 publickey
AllowedIPs = 10.100.1.2/24
PersistentKeepalive = 15
```

# 启动
```shell
wg-quick up wg0
wg-quick down wg0
```


# 客户端
```
wg genkey | sudo tee /etc/wireguard/privatekey-wg0 | wg pubkey | sudo tee /etc/wireguard/publickey-wg0 #客户端和服务生成的密钥指令一样

[Interface]
PrivateKey = CLIENT_PRIVATE_KEY  #客户端生成的私钥
Address = 10.100.1.2/32 #改为想给本机分配的ip
DNS = 114.114.114.114   #dns地址

[Peer]
PublicKey = SERVER_PUBLIC_KEY  #服务器端生成的公钥
Endpoint = SERVER_IP_ADDRESS:51800  #服务器的公网IP和端口
AllowedIPs = 10.100.1.0/24	#这里是限制只有10.100.1 网段的请求走WireGuard,不影响其它应用上网,设为0.0.0.0/0则是所有请求均走WireGuard
PersistentKeepalive = 120	#握手时间，每隔120s ping一次客户端

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

