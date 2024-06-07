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
```

# 启动
```shell
wg-quick up wg0
wg-quick down wg0
```


# 客户端
```
[Interface]
PrivateKey = CLIENT_PRIVATE_KEY  #客户端生成的私钥
Address = 10.100.1.2/32 #改为想给本机分配的ip

[Peer]
PublicKey = SERVER_PUBLIC_KEY  #服务器端生成的公钥
Endpoint = SERVER_IP_ADDRESS:51800  #服务器的公网IP和端口
AllowedIPs = 10.100.1.0/24	#这里是限制只有10.100.1 网段的请求走WireGuard,不影响其它应用上网,设为0.0.0.0/0则是所有请求均走WireGuard
PersistentKeepalive = 120	#握手时间，每隔120s ping一次客户端

```