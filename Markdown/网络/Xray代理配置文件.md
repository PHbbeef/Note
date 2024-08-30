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

```
监听本地端口10808。

监听端口收到数据转到路由模块，路由模块在这里处理分流该走哪。

`type`路由标识，不能重复
```