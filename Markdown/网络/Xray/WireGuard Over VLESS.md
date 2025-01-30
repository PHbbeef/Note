## 实现
+ 国内机器 A，公网 IP 1.3.5.7，分配内网 IP 10.10.0.1
+ 国外机器 B，公网 IP 2.4.6.8，分配内网 IP 10.10.20.1


## VLESS配置文件
在国外机器 B 上建立 VLESS 入站，在 2.4.6.8 的 52220 端口 上接收 VLESS TCP 流量。
```json
{
    "log": {
        "loglevel": "debug"
    },
    "dns": {
        "servers": [
            "1.1.1.1",
            "8.8.8.8",
            "8.8.4.4"
        ]
    },
    "inbounds": [
        {
            "listen": "0.0.0.0",
            "port": 52220,
            "protocol": "vless",
            "settings": {
                "clients": [
                    {
                        "id": "c71c5890-56dd-4f32-bb99-3070ec2f20fa"
                    }
                ],
                "decryption": "none"
            },
            "streamSettings": {
                "network": "tcp"
            }
        }
    ],
    "outbounds": [
        {
            "protocol": "freedom",
            "settings": {
                "domainStrategy": "UseIP"
            },
            "tag": "free"
        }
    ],
    "routing": {
        "rules": []
    }
}
```

在国内机器 A 上建立 VLESS 出站，入站使用 dokodemo-door 协议，添加一个 UDP 转发，监听 A 上的 10000 端口的 UDP 流量，将该流量通过 VLESS 出站，转发到 Server 端的 127.0.0.1:10000 上。
```json
{
    "log": {
        "loglevel": "debug"
    },
    "inbounds": [
        {
            "listen": "127.0.0.1",
            "port": 10000,
            "protocol": "dokodemo-door",
            "settings": {
                "address": "127.0.0.1",
                "port": 10000,
                "network": "udp"
            },
            "tag": "wg"
        }
    ],
    "outbounds": [
        {
            "protocol": "freedom",
            "settings": {}
        },
        {
            "protocol": "vless",
            "settings": {
                "vnext": [
                    {
                        "address": "2.4.6.8",
                        "port": 52220,
                        "users": [
                            {
                                "id": "c71c5890-56dd-4f32-bb99-3070ec2f20fa",
                                "encryption": "none"
                            }
                        ]
                    }
                ]
            },
            "streamSettings": {
                "network": "tcp"
            },
            "tag": "vless-tunnel"
        }
    ],
    "routing": {
        "rules": [
            {
                "type": "field",
                "inboundTag": [
                    "wg"
                ],
                "outboundTag": "vless-tunnel"
            }
        ]
    }
}
```
至此，我们可以将流向国内机器 A 127.0.0.1:10000 的 UDP 流量，经由 VLESS 隧道，转发到国外机器 B 的 127.0.0.1:10000 上。


## WireGuard 配置
在国内机器 A 上，添加一个 Interface，设置 Endpoint 为 127.0.0.1:10000，流入 VLESS 的入站。
```
[Interface]
Address = 10.10.0.1
DNS = 1.1.1.1
PrivateKey = 私钥
ListenPort = 51820
MTU = 1420

[Peer]
PublicKey = B 的公钥
AllowedIPs = 10.10.20.1
Endpoint = 127.0.0.1:10000
PersistentKeepalive = 20
```

在国外机器 B 上，也添加 Interface，监听在 10000 端口，接收 VLESS 转发过来的流量。
```
[Interface]
Address = 10.10.20.1
ListenPort = 10000
PrivateKey = 私钥
MTU = 1420

[Peer]
PublicKey = A 的公钥
AllowedIPs = 10.10.0.1
```

## 相关问题
1
```
如果 wireguard 中传输 tcp 流量，就会是 tcp (chrome) - udp (wireguard) - tcp (vless)，容易出现双层拥塞控制雪崩。
更好的做法是 wireguard over udp (ss2022)，这样就是 tcp (chrome) - udp (wireguard) - udp (ss2022)，这样只有一层拥塞控制。
```