
## 基本配置
分流用到以下三个
|类型|说明|
|-|-|
|inbounds|入站|
|outbounds|出战|
|routing|路由|


|类型|说明 |
|---|---|
|direct|直连|
|blackhole|黑洞|
|Freedom|出站协议可以用来向任意网络发送（正常的） TCP 或 UDP 数据。|

`balancerTag` 和 `outboundTag` 须二选一。当同时指定时，`outboundTag` 生效。

+ 监听本地端口10808。
+ 监听端口收到数据转到路由模块，路由模块在这里处理分流该走哪。
+ `type`路由标识，不能重复

### 数据包处理流程
```json
{
 "inbounds": [  //  入站，通过socks5代理协议
    {
      "tag": "socks",           // 标签
      "port": 10808,            // 端口
      "listen": "127.0.0.1",    //监听地址
      "protocol": "socks",      // 连接协议名称
    }
  ],
  "outbounds": [    // 出战
    {
      "tag": "proxy",
      "protocol": "vmess",
    }
    {
        "tag": "direct",
        "protocol": "freedom",
    },
    {
        "tag": "block",
        "protocol" "blackhole"
    }
  ],
  "routing": {                          // 路由
    "domainStrategy": "IPIfNonMatch",   // 域名解析策略，根据不同的设置使用不同的策略
    "rules": [
        {
            "ip" ["geoid:cn"],
            "outboundTag":"direct"
        },
        {
            "domain"["geosite:cn"], // 所有中国域名
            "outboundTag":"direct"  // 对应一个 outbound 的标识
        },
        {
            "type": "field",
            "ip": [
                "geoip:private"     // geoip:private包含内网IP
            ],
            "outboundTag": "freedom",   // freedom 可以访问服务器内网设备

        }
    ]
  }
}
```