## 安装和配置 官方的安装有详细说明

```bash
#创建用户
useradd steam
#设置密码
passwd steam


#Debian或Ubuntu下载依赖
apt-get install lib32gcc-s1

#切换至steam用户至steam家目录(home)
#为SteamCMD创建一个目录
mkdir ~/Steam && cd ~/Steam
#下载SteamCMD解压并运行
wget https://steamcdn-a.akamaihd.net/client/installer/steamcmd_linux.tar.gz
#注：如果在其他文件目录下无法运行，把SteamCMD解压到家目录下运行(home)
./steamcmd.sh
```

## 饥荒搭建

```C++
./dontstarve_dedicated_server_nullrenderer -console -cluster MyDediServer -shard Master

./dontstarve_dedicated_server_nullrenderer -console -cluster MyDediServer -shard Caves
```

## 其他说明

Q1：SteamCMD的存档在哪？
Q2：下载的服务端在什么位置？

A1：家目录下注意隐藏文件(home)
A2：注意程序目录结构，steamapps/common
