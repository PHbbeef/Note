# OpenVPN搭建

## 基本服务端配置文件
```
local 192.168.0.2        # 监听本地接口192.168.0.2
port 1194                       # 端口1194
proto udp                      # 使用 UDP。TCP会有严重的性能问题。
dev tap                           # 将以太网二层报文装入DTLS会话。 注：如果要改为桥接模式注释到dev tun 改为dev tap
ca /etc/ca.crt                # CA证书
cert /etc/server.crt   # 证书
key /etc/server.key.txt  # 私钥
dh /etc/dh.pem          # Deffie Hellman参数
auth SHA512               # 使用SHA512认证，不然默认SHA1。
server 192.168.199.0 255.255.255.0   # 分配网段192.168.199.0/24
push "route 172.21.0.0 255.255.255.0"   #服务端向客户端推送vpn服务端内网网段的路由配置
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
./easyrsa build-server-full server nopass   #生成服务端无密码证书

#生成客户端证书
./easyrsa build-client-full client nopass   #生成无密码客户端证书

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

route-nopull #如果加这一行任何代理都不会走；不走任何代理

#注：Linux设置全局代理连接服务器的这个ip要排除掉否则连不上
route 47.101.145.0 255.255.255.0 net_gateway
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

### 配置文件

#### 账户密码登录
配置文件开启账户密码
```bash
$ vim /etc/openvpn/server.conf  # 打开server.conf配置文件编辑


# use username and password login
# 新加此行，开启密码验证脚本
auth-user-pass-verify /etc/openvpn/checkpsw.sh via-env
# 加上client-cert-not-required代表只使用用户密码方式验证登录，不加则代表需要证书和用户名密码双重验证登录
client-cert-not-required
# 新加此行，使用客户提供的UserName作为Common Name
username-as-common-name
# 该指令提供对OpenVPN使用外部程序和脚本的策略级别的控制。较低的 水平 值更具限制性，较高的值更宽松。级别设置 
# 0- 完全不调用外部程序。
# 1- （默认）仅调用内置可执行文件，例如ifconfig，ip，route或netsh。
# 2- 允许调用内置的可执行文件和用户定义的脚本。
# 3- 允许通过环境变量将密码传递给脚本（可能不安全）。
# 特别注意如果没有这个配置项会导致服务端校验密码时无法获取到密码，导致校验失败
script-security 3
```

验证密码脚本
```bash
#!/bin/sh
###########################################################
# checkpsw.sh (C) 2004 Mathias Sundman <mathias@openvpn.se>
#
# This script will authenticate OpenVPN users against
# a plain text file. The passfile should simply contain
# one row per user with the username first followed by
# one or more space(s) or tab(s) and then the password.
###########################################################

PASSFILE="/etc/openvpn/user_passwd.txt"
LOG_FILE="/var/log/openvpn/openvpn-login.log"
TIME_STAMP=`date "+%Y-%m-%d %T"`

if [ ! -r "${PASSFILE}" ]; then
  echo "${TIME_STAMP}: Could not open password file \"${PASSFILE}\" for reading." >> ${LOG_FILE}
  exit 1
fi

CORRECT_PASSWORD=`awk '!/^;/&&!/^#/&&$1=="'${username}'"{print $2;exit}' ${PASSFILE}`

if [ "${CORRECT_PASSWORD}" = "" ]; then 
  echo "${TIME_STAMP}: User does not exist: username=\"${username}\", password=\"${password}\"." >> ${LOG_FILE}
  exit 1
fi

if [ "${password}" = "${CORRECT_PASSWORD}" ]; then 
  echo "${TIME_STAMP}: Successful authentication: username=\"${username}\"." >> ${LOG_FILE}
  exit 0
fi

echo "${TIME_STAMP}: Incorrect password: username=\"${username}\", password=\"${password}\"." >> ${LOG_FILE}
exit 1
```

```bash
$ vim /etc/openvpn/user_passwd.txt # 编辑账号密码文件，添加以下内容
eddieeo c123456@
```

客户端配置
```bash
# 注释掉客户端密钥认证方式
;cert laptop.crt
;key laptop.key

# 新增账号/密码验证方式
auth-user-pass
```

## 其他配置

win10服务端搭建，客户端可以访问服务端内网设备
```C++
// 开启网卡共享
/*
假设服务端使用的网卡是 SSTAP，服务器内网网段网卡是 以太网。注：内网网段网卡共享SSTAP
*/
网卡属性 -> 共享 -> internet 连接共享

// ******************* 此方法不可用 *******************
/* 
Linux 下可以是使用iptable，win下面使用路由表的配置。具体如下
假设服务器的内网子网为192.168.1.0/24，而OpenVPN服务器在该内网中的IP地址为10.8.0.1。
*/
route -p add 192.168.1.0 mask 255.255.255.0 10.8.0.1    // -p 添加永久路由
```


## 改为桥接
[OpenVPN文档](https://openvpn.net/community-resources/ethernet-bridging/)
注：客户端要改为 dev tap
```C++
// Linux 配置

brctl addbr br0             //创建网桥
brctl addif br0 eth0        //内部网络加入网桥

server-bridge [gw] [mask] [start-IP] [end-IP]   //OpenVPN服务器ip分配设置  

brctl addif br0 tap0    //OpenVPN的网卡加入


// ############################# 分割线 #############################

// Windows 配置

控制面板\网络和 Internet\网络连接，选中两个网卡添加桥接；修改为静态ip    //在 Windows 里面设置网卡的桥接；这里记住OpenVPN的网卡名，这里为tap-bridge
打开 dev tap 注释掉 dev tun     //使得 OpenVPN Server 工作在桥接模式下
找到 dev-node 那一行，打开注释。写成：dev-node tap-bridge   //就是之前在 Windows 底下修改的那个 Tap 网卡的名字。前面我提到修改为 tap-bridge
注释掉 server 开头的行替换为 server-bridge [gw] [mask] [start-IP] [end-IP] 
```