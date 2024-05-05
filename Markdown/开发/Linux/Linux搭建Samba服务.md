## 安装
```shell
apt-get update

apt-get install samba

whereis samba

#配置文目录
vim /etc/samba/smb.conf
```


## 配置文件信息
```shell
[data]
 # 文件描述
 comment = Samba on Armbian
 # 目录文件
 path = /home/armbian/data
 # 是否只读
 read only = no
 # 是否可流量
 browsable = yes
```

## 启动
```shell
# 重启
service smbd restart
```

```shell
# 设置密码
smbpasswd -a root
```