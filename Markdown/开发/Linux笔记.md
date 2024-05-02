# 目录结构

```bash
#源文件
/etc/apt

#网络配置（网关，dns，掩码）
/etc/network/interfaces
#静态配置入校
auto enp0s3
iface enp0s3 inet static
address 192.168.2.2
netmask 255.255.255.0
gateway 192.168.2.2
dns-nameservers 8.8.4.4 8.8.8.8
```

# 环境变量

Linux是一个多用户操作系统，每个用户都有自己专有的运行环境。用户所使用的环境由一系列变量所定义，这些变量被称为环境变量。系统环境变量通常都是大写的。

在终端运行可执行程序时是首先会在当前目录查找，如果找不到则会在环境变量中去找。

*   环境变量告诉可执行程序的绝对路径，以便可以不在当前目录执行程序。

```C++
// 1、bash:useradd:command not found。如果安装后还是提示错误配置环境变量
PATH=$PATH:/bin:/usr/sbin
```

## 常用指令

```shell
#添加环境变量值
export DOTNET_ROOT=$HOME/dotnet
PATH=$PATH:$HOME/dotnet
sudo vim /etc/bash.bashrc   #永久添加环境 
source /etc/bash.bashrc # 更新环境配置

#删除环境变量（删除PATH变量）
unset PATH

#查看PATH变量的值（或路径）
echo $PATH

#查看打印命令
 printenv
 env
```

# 加入sudo用户组

```bash
#切换root用户
su -

usermod -aG sudo [要加入的用户名]
```

# 文件同步Rsync

```bash
#配置环境
vi /etc/rsyncd.conf

uid=root
gid=root
max connections=4
log file=/var/log/rsyncd.log
pid file=/var/run/rsyncd.pid
lock file=/var/run/rsyncd.lock
secrets file=/etc/rsyncd.passwd
hosts deny=172.16.78.0/22

[steam]
comment= 连接stema用户  #注释（连接时不加连接用户会显示用户和注释；work@192.168.99::）
path=/home/steam/       #客户端访问的文件路径
read only = no
exclude=test
auth users=work         #连接用户（这里要和rsyncd.passwd对应；work@192.168.99）


#其他
rsync --daemon  #(启动服务；端口是873)
rsync --list-only work@192.168.1.99::   #(列出文件而不是复制它们)

#常用同步指令（无需再输入密码）
rsync -rtvzP --password-file=rsyncd.txt work@192.168.1.99::
```

# 其他指令

## davfs2

```bash
# 使用坚果云的webdab报错：the server does not support WebDAV
# 修改 /etc/davfs2/davfs2.conf 配置文件取消掉注释
ignore_dav_header 1
```

## 管道

```bash
#配合管道符筛选指定名
dpkg -l |grep vim
```

## 挂载硬盘

```C++
// 查看硬盘信息
lsblk

// 格式化指定硬盘
sudo mkfs -t ext4 /dev/sda1

// 挂载指定目录
sudo mount /dev/sda1 /data

// 查看挂载硬盘信息
df -h
```

## 添加开机自启动

```C++
vi /etc/systemd/system/qbittorrent.service //使用vi设置开机自启服务

  
[Unit]
Description=qBittorrent Daemon Service
After=network.target
[Service]
User=root
ExecStart=/usr/bin/qbittorrent-nox
ExecStop=/usr/bin/killall -w qbittorrent-nox
[Install]
WantedBy=multi-user.target


systemctl daemon-reload //更新服务
service qbittorrent start //启动qb
service qbittorrent status //查看qb状态
systemctl enable qbittorrent //设置QB开机启动
service qbittorrent stop //关闭qb
systemctl disable qbittorrent //关闭开机自启
```

