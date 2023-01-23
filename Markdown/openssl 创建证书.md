# 1. 创建根证书

## 模拟 CA 机构
```sh
# 创建 CA 私钥
openssl genrsa -out ca.key 2048
# 制作 CA 公钥（证书）
openssl req -new -x509 -days 3650 -key ca.key -out ca.crt
```

# 2. 签发证书
```sh
# 创建密钥（私钥/公钥）
openssl genrsa -out a.pem 1024
openssl rsa -in a.pem -out a.key
# 生成签发请求
openssl req -new -key a.pem -out a.csr
# 使用 CA 证书进行签发
openssl x509 -req -sha256 -in a.csr -CA ca.crt -CAkey ca.key -CAcreateserial -days 3650 -out a.crt
# 验证签发证书是否正确
openssl verify -CAfile ca.crt a.crt
```

# 其他
原文[地址](https://blog.csdn.net/Jacoh/article/details/126213598)