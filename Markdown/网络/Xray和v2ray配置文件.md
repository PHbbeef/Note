## 基本介绍
两个内核的配置文件大致相同，有些地方有细微区别

入站(Inbound)

分客户端和服务端那么入站是用在服务端

Shadowsocks协议配置

详细配置参见官网
```json
// Xray内核和v2ray内核配置基本相同
{
  "inbounds": [
    {
      "port": 1024, // 监听端口
      "protocol": "shadowsocks",
      "settings": {
        "method": "aes-128-gcm",
        "ota": true, // 是否开启 OTA
        "password": "sspasswd"
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

// ################################# 分割线 ################################# 
// socks5 配置
{
	"listen": "0.0.0.0",
	"port": 5555,
	"protocol": "socks",
	"settings":{    //注意这里密码是在`settings`
		"auth": "password",
		"accounts":[
			{
				"user": "1234567890",
				"pass": "1234567890"
			}
		]
	}
}
```


出站(Outbound)