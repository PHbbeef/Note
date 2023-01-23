## 文件信息
| 名称 | 作用 |
|--|--|
| frps.ini | 服务端配置 |
| frps_full.ini | 序和配置 |
|||
| frpc.ini | 客户端配置 |
| frpc_full.ini | 序和配置 |

### 配置服务器
```
[common]
bind_port = 7000    #客户端链接服务器的端口
token = 101111      #设置密钥，客户端也需要设置，为了安全
vhost_http_port = 7080      #虚拟主机端口，可以通过这个端口访问不通子域的客户端
dashboard_port = 7070       #服务端web端口
dashboard_user = admin      #web用户
dashboard_pwd = admin       #web密码
```
#### 启动服务器
```
./frps -c ./frps.ini
```

### 客户端配置
可以配置多个
```
[common]
server_addr = 127.0.0.1     #服务端IP（IP或域名）
server_port = 7000          #服务端监听的端口,客户端连接使用
token = 101111              #连接服务器的密钥，与服务器保持一致

[Terraria]                  #名称（随便填）
type = tcp                  #传输协议
local_ip = 127.0.0.1        #客户端IP，如果多个可指定其中一个
local_port = 7777           #内网映射端口
remote_port = 8888          #外网访问端酒

[MC]                 
type = udp                  
local_ip = 127.0.0.1      
local_port = 5023           
remote_port = 5023 
```
#### 启动客户端
```
./frpc -c ./frpc.ini
```