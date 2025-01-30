`config.json`用到三个配置文件
|||
|-|-|
|inbounds|入站|
|outbounds|出战|
|routing|路由|

### 数据包处理流程
```json
{
 "inbounds": [
    {
      "tag": "socks",
      "port": 10808,
      "listen": "127.0.0.1",
      "protocol": "socks",
    }
  ],
  "outbounds": [
    {
      "tag": "proxy",
      "protocol": "vmess",
      //(省略vmess详细配置)
    }
    {
        "tag": "direct",
        "protocol": "freedom",
        //freedom直连
    },
    {
        "tag": "block",
        "protocol" "blackhole"
        //blackhole黑洞(丢弃数据包)
    }
  ],
  "routing": {
    "domainStrategy": "IPIfNonMatch",
    "rules": [
        {
            "ip" ["geoid:cn"],
            "outboundTag":"direct"
            //匹配策略
        },
        {
            //匹配策略
            "domain"["geosite:cn"],
            "outboundTag":"direct"
        },
        {
            "type": "field",
            "ip": [
                "geoip:private"
                // geoip:private包含内网IP
            ],
            "outboundTag": "blocked",

        }
    ]
  }
}
```

### 入站和出战配置文件
```json
{
  "inbounds": [
    {
      "listen": "127.0.0.1",
      "port": 1080,
      "protocol": "协议名称",
      "settings": {},
      "streamSettings": {},
      "tag": "标识",
      "sniffing": {
        "enabled": true,
        "destOverride": ["http", "tls"]
      },
      "allocate": {
        "strategy": "always",
        "refresh": 5,
        "concurrency": 3
      }
    }
  ]
}

*************************************分割线*************************************
出站

{
  "outbounds": [
    {
      "sendThrough": "0.0.0.0",
      "protocol": "协议名称",
      "settings": {},
      "tag": "标识",
      "streamSettings": {},
      "proxySettings": {
        "tag": "another-outbound-tag"
      },
      "mux": {}
    }
  ]
}

```


```
监听本地端口10808。

监听端口收到数据转到路由模块，路由模块在这里处理分流该走哪。

`type`路由标识，不能重复
```


## 反向代理
使用反向代理使用家宽IP解决流媒体限制。或是内网穿透家里的设备


客户端
```json
{
    "log": {
        "loglevel": "info", 
        "access": "", 
        "error": ""
    }, 
    "reverse": {
        "bridges": [
            {
                "tag": "bridge", 
                "domain": "A 和 B 反向代理通信的域名，可以自己取一个，可以不是自己购买的域名，但必须跟下面 B 中的 reverse 配置的域名一致"
            }
        ]
    }, 
    "outbounds": [
        {
            "tag": "tunnel", 
            "protocol": "vmess", 
            "settings": {
                "vnext": [
                    {
                        "address": "服务端ip或者域名", 
                        "port": 暴露的端口, 
                        "users": [
                            {
                                "id": "服务端uuid", 
                                "alterId": 64
                            }
                        ]
                    }
                ]
            }
        }, 
        {
            "protocol": "freedom", 
            "settings": {
                "redirect": "内网nas的地址和端口"
            }, 
            "tag": "out"
        }
    ], 
    "routing": {
        "rules": [
            {
                "type": "field", 
                "inboundTag": [
                    "bridge"
                ], 
                "domain": [
                    "full:和上面的域名保持一致"
                ], 
                "outboundTag": "tunnel"
            }, 
            {
                "type": "field", 
                "inboundTag": [
                    "bridge"
                ], 
                "outboundTag": "out"
            }
        ]
    }
}
```

服务端
```json
{
  "log":{
    "loglevel": "info",
    "access": "日志放的地址",
    "error": "日志放的地址"
  },
  "reverse":{
    "portals":[
      {
        "tag":"portal",
        "domain":"必须和上面 A 设定的域名一样"
      }
    ]
  },
  "inbounds":[
    {
      "tag":"external",
      "port":接收C发来的请求的端口,可用nginx做转发,
      "protocol":"dokodemo-door",
        "settings":{
          "address":"nas内网地址",
          "port":nas内网端口,
          "network":"tcp"
        }
    },
    {
      "tag": "tunnel",
      "port":A主动连接B用的端口,
      "protocol":"vmess",
      "settings":{
        "clients":[
          {
            "id":"服务器端配置的uuid",
            "alterId":64
          }
        ]
      }
    }
  ],
  "routing":{
    "rules":[
      {
        "type":"field",
        "inboundTag":[
          "external"
        ],
        "outboundTag":"portal"
      },
      {
        "type":"field",
        "inboundTag":[
          "tunnel"
        ],
        "domain":[
          "full:和上面一致的域名"
        ],
        "outboundTag":"portal"
      }
    ]
  }
}
```