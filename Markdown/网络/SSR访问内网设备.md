## 需要注意以下：
* 如果连接成功无法访问内网设备，检查软件是否代理内网网段

## 安装和配置文件
+ 这里使用的是[Shadowsocks-Rust](https://github.com/shadowsocks/shadowsocks-rust)
+ 下载自己对应的版本，解压
+ 在当前目录新建一个 <font color="red">config.json</font>
```
{
  "server": "[::]",
  "server_port": 59876,
  "timeout": 60,
  "method": "aes-128-gcm",
  "password": "1234567890",
  "mode": "tcp_and_udp"
}
```
### 配置说明
+ 监听IPV6
+ 监听端口
+ 超时时间
+ 加密算法
+ 密码
+ 监听TCP和UDP
