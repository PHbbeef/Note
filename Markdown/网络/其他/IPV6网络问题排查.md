
## 解决思路
```bash
# 查看IPv6地址
ip -6 addr show

# 查看IPv6路由
ip -6 route show

# 查看默认网关
ip -6 route | grep default

# 查看邻居缓存
ip -6 neigh show

# 查看IPv6 DNS配置
cat /etc/resolv.conf

# 跟踪到目标的路由
traceroute6 -n v6hk.drlihui.eu.org

# 检查是否有冲突路由
ip -6 route show | grep "2401:b60"
```