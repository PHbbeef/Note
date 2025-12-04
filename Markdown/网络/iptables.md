```bash
# 自动将数据包的源IP地址替换为出口网卡的IP地址
# 只能在 POSTROUTING 链中使用
# 主要实现网络地址转换，让内网设备通过网关访问外网
iptables -t nat -I POSTROUTING -j MASQUERADE


# 1. 指定出口网卡
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

# 2. 指定源地址范围
iptables -t nat -A POSTROUTING -s 192.168.1.0/24 -o eth0 -j MASQUERADE

# 3. 指定端口范围（可选）
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE --to-ports 1024-65535
```