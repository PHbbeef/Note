## 基本介绍
Xray[下载地址](https://github.com/XTLS/Xray-core)

### Hysteria2配置
Hysteria内核[下载地址](https://github.com/apernet/hysteria)
```yaml
#################### 服务端 ####################

#vim /etc/hysteria/config.yaml

#acme:
#  domains:
#    - test.drlihui.eu.org        # 域名（解析到自己服务IP）
#  email: your@email.com   # 邮箱，格式正确即可;这个邮箱随机

# 证书文件路径（没有域名）
tls:
  cert: /etc/hysteria/server.crt
  key: /etc/hysteria/server.key

auth:
  type: password
  password: 1234567890

masquerade:
  type: proxy
  proxy:
    url: https://bing.com
    rewriteHost: true


#################### 客户端 ####################
server: 46.17.43.105:8443   #服务器IP和端口
auth: 1234567890            #密码
 
#bandwidth:
#  up: 20 mbps
#  down: 100 mbps
  
tls:
  sni: bing.com  # 若无域名，请改为 bing.com
  insecure: true    # 若无域名，需要改参数为 true
 
socks5:
  listen: 127.0.0.1:1080
http:
  listen: 127.0.0.1:8080
```

### SSR配置
```json
{
  "inbounds": [
    {
      "port": 1024, // 监听端口
      "protocol": "shadowsocks",
      "settings": {
        "method": "aes-128-gcm",    //加密方式
        "ota": true, // 是否开启 OTA
        "password": "1234567890",   //密码
		"network": "tcp,udp"        //支持TCP,UDP
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "freedom",  
      "settings": {}
    }
  ]
}
```
### VMess+WS+CDN配置
```json
{
  "log": null,
  "routing": {
    "rules": [
      {
        "inboundTag": [
          "api"
        ],
        "outboundTag": "api",
        "type": "field"
      },
      {
        "ip": [
          "geoip:private"
        ],
        "outboundTag": "blocked",
        "type": "field"
      },
      {
        "outboundTag": "blocked",
        "protocol": [
          "bittorrent"
        ],
        "type": "field"
      }
    ]
  },
  "dns": null,
  "inbounds": [
    {
      "listen": "127.0.0.1",
      "port": 62789,
      "protocol": "dokodemo-door",
      "settings": {
        "address": "127.0.0.1"
      },
      "streamSettings": null,
      "tag": "api",
      "sniffing": null
    },
    {
      "listen": null,
      "port": 2053,     //CDN HTTPS端口
      "protocol": "vmess",
      "settings": {
        "clients": [
          {
            "id": "c71c17ee-38a9-4ac2-9bc1-175fd0324700",
            "alterId": 0
          }
        ],
        "disableInsecureEncryption": false
      },
      "streamSettings": {
        "network": "ws",
        "security": "tls",
        "tlsSettings": {
          "serverName": "www.drlihui.eu.org",
          "certificates": [
            {
                "certificateFile": "/root",     //公钥路径
                "keyFile": "/root"              //密钥路径

            }
          ]
        },
        "wsSettings": {
          "path": "/",
          "headers": {
            "Host": "www.drlihui.eu.org"
          }
        }
      },
      "tag": "inbound-2053",
      "sniffing": {
        "enabled": true,
        "destOverride": [
          "http",
          "tls"
        ]
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "freedom",
      "settings": {}
    },
    {
      "protocol": "blackhole",
      "settings": {},
      "tag": "blocked"
    }
  ],
  "transport": null,
  "policy": {
    "system": {
      "statsInboundDownlink": true,
      "statsInboundUplink": true
    }
  },
  "api": {
    "services": [
      "HandlerService",
      "LoggerService",
      "StatsService"
    ],
    "tag": "api"
  },
  "stats": {},
  "reverse": null,
  "fakeDns": null
}
```