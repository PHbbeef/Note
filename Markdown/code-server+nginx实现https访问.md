# code-server+nginx实现https访问


## nginx
### 生成证书
<font color="red">注意更改自己的信息</font>
```sh
# vim req.cnf
```
```
# 定义输入用户信息选项的"特征名称"字段名，该扩展字段定义了多项用户信息。
distinguished_name = req_distinguished_name

# 生成自签名证书时要使用的证书扩展项字段名，该扩展字段定义了要加入到证书中的一系列扩展项。
x509_extensions = v3_req

# 如果设为no，那么 req 指令将直接从配置文件中读取证书字段的信息，而不提示用户输入。
prompt = no


[req_distinguished_name]
#国家代码，一般都是CN(大写)
C = CN
#省份
ST = Beijing
#城市
L = Beijing
#企业/单位名称
O = oyz
#企业部门
OU = oyz
#证书的主域名
CN = 192.168.8.190

##### 要加入到证书请求中的一系列扩展项 #####
[v3_req]
keyUsage = critical, digitalSignature, keyAgreement
extendedKeyUsage = serverAuth
subjectAltName = @alt_names


[ alt_names ]
DNS.1 = oyz.com
DNS.2 = *.oyz.com
DNS.3 = www.flymot.com
DNS.4 = *.oyz.com
IP.1 = 192.168.8.190
IP.2 = 192.168.1.190
```

```sh
# openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /usr/local/nginx/ssl/private.key -out /usr/local/nginx/ssl/nginx.crt -config req.cnf -sha256
```
```
req          大致有3个功能：生成证书请求文件、验证证书请求文件和创建根CA
-x509        说明生成自签名证书
-nodes       openssl req在自动创建私钥时，将总是加密该私钥文件，并提示输入加密的密码。可以使用"-nodes"选项禁止加密私钥文件。
-days        指定所颁发的证书有效期。
-newkey      实际上，"-x509"选项和"-new"或"-newkey"配合使用时，可以不指定证书请求文件，它在自签署过程中将在内存中自动创建证书请求文件
              "-newkey"选项和"-new"选项类似，只不过"-newkey"选项可以直接指定私钥的算法和长度，所以它主要用在openssl req自动创建私钥时。
rsa:2048     rsa表示创建rsa私钥，2048表示私钥的长度。
-keyout      指定私钥保存位置。
-out         新的证书请求文件位置。
-config      指定req的配置文件，指定后将忽略所有的其他配置文件。如果不指定则默认使用/etc/pki/tls/openssl.cnf中req段落的值
```

### nginx使用证书
```
 server {
   #监听7070端口
    listen 7070 ssl;
    #你的域名
    server_name 0.0.0.0;
    #ssl证书的pem文件路径
    ssl_certificate  /usr/local/test/ssl/cert.crt;
    #ssl证书的key文件路径
    ssl_certificate_key /usr/local/test/ssl/cert.key;

    location / {
        proxy_pass   http://127.0.0.1:8080/;
        proxy_set_header Host $host;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection upgrade;
        proxy_set_header Accept-Encoding gzip;
    }
 }
```