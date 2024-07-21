# OpenVPN搭建

## 基本服务端配置文件
```
local 192.168.0.2        # 监听本地接口192.168.0.2
port 1194                       # 端口1194
proto udp                      # 使用 UDP。TCP会有严重的性能问题。
dev tap                           # 将以太网二层报文装入DTLS会话。
ca /etc/ca.crt                # CA证书
cert /etc/server.crt   # 证书
key /etc/server.key.txt  # 私钥
dh /etc/dh.pem          # Deffie Hellman参数
auth SHA512               # 使用SHA512认证，不然默认SHA1。
server 192.168.199.0 255.255.255.0   # 分配网段192.168.199.0/24
push "dhcp-option DNS 192.168.199.1"       # 向客户端推送DNS服务器IP
client-to-client             # 允许客户端之间互通
keepalive 10 120         # 保持连接
tls-crypt-v2 /etc/server.key.txt # 使用该密钥提供控制平面的额外加密
```

## 证书
注：这是在Windows运行和Linux下有指令上的区别
```shell
# 初始化证书
#会在当前目录创建kpi文件夹
./easyrsa init-pki 

#生成ca证书
./easyrsa build-ca nopass

#生成服务端证书
./easyrsa build-server-full server nopass

#生成客户端证书
./easyrsa build-client-full client nopass

#生成DH密钥交换协议
./easyrsa gen-dh
```

## 修改服务端配置文件
```
复制server.ovpn文件至C:\Program Files\OpenVPN\config目录,修改如下选项:
a)端口:(公网需要对应开通此端口)port udp 1194
b)协议文件名:dh dh2048.pem修改为dh dh.pem
c)运行多用户使用同一客户端证书:;duplicate-cn取消注释(前面;号删除)修改为duplicate-cn
d)注释掉此行(前面加#号): tls-auth ta.key 0修改为#tls-auth ta.key 0


将服务证书,服务key,ca证书,dh文件复制到文件夹C:\Program Files\OpenVPN\config下
```

## 客户端配置文件
```
1.配置客户端文件
客户端配置文件client.ovpn,模板在C:\Program Files\OpenVPN\sample-config,复制该文件
到C:\Program Files\OpenVPN\config目录下,修改客户端配置如下:
a)修改连接服务器地址: remote my-server-1 1194修改为remote 服务器公网ip 1194
b)注释掉此行(前面加#号): tls-auth ta.key 1修改为# tls-auth ta.key 1


生成的客户端证书和配置文件,客户端需要的一共有5个文件:ca.crt、client.crt、client.key、
ta.key(如果不开启tls-auth,则无需该文件)、client.ovpn。为了避免文件太多,管理不便!需要
将客户端证书合并到配置文件中。具体方法如下。
注：如果要使用，注释掉配置文件中ca，cert，key等
<ca>
ca.crt文件内容
</ca>
<cert>
client.crt文件内容
</cert>
<key>
client.key文件内容
</key>
key-direction 1
<tls-auth>
ta.key文件内容
</tls-auth>
```

## 其他相关配置
### 配置静态代理
```shell
# 修改服务端配置文件
# 注释掉服务端配置文件，正常退出会在ipp.txt文件出现数据
ifconfig-pool-persist ipp.txt
```

### 指定代理
```shell
route 192.168.1.0 255.255.0.0 net_gateway #不走代理
route 0.0.0.0 0.0.0.0 vpn_gateway #走代理

route-nopull #如果加这一行任何代理都不会走
```

### 配置开机自启
```shell
# 注：--cd命令
/usr/sbin/openvpn --cd /etc/openvpn/server/ --config /etc/openvpn/server/server.ovpn
```

### 使用TCP报错
```shell
# 服务端这个配置文件注释掉
explicit-exit-notify 1
```
