## 基本介绍
Xray[下载地址](https://github.com/XTLS/Xray-core)

### Hysteria2配置
Hysteria内核[下载地址](https://github.com/apernet/hysteria)

```bash
#生成证书

openssl req -x509 -nodes -newkey ec:<(openssl ecparam -name prime256v1) -keyout /etc/hysteria/server.key -out /etc/hysteria/server.crt -subj "/CN=bing.com" -days 36500 && sudo chown hysteria /etc/hysteria/server.key && sudo chown hysteria /etc/hysteria/server.crt
```
```yaml
#################### 服务端 ####################

#vim /etc/hysteria/config.yaml

# 有域名
acme:
  domains:
    - test.drlihui.eu.org        # 域名（解析到自己服务IP）
  email: your@email.com   # 邮箱，格式正确即可;这个邮箱随机

# *****这个两个↑↓选择其中一个

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
  listenHTTPS: :443 #监听443端口；这样访问服务端443端口会跳转bing网址
  forceHTTPS: true  #是否强制使用 HTTPS


#################### 客户端 ####################
server: 46.17.43.105:8443   #服务器IP和端口
auth: 1234567890            #密码
 
bandwidth:  #根据自己的带宽调整
  up: 20 mbps
  down: 100 mbps
  
tls:
  sni: bing.com  # 若无域名，请改为 bing.com
  insecure: true    # 若无域名，需要改参数为 true
 
socks5:
  listen: 127.0.0.1:1080
http:
  listen: 127.0.0.1:8080
```
其他问题
```
1.服务端出现`server up and running` 运行成功
2.q服务端启动失败 a查看服务端端口受防火墙影响。证书需要用到80和443端口
3.`journalctl -u hysteria.service`可以查看全部日志信息
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