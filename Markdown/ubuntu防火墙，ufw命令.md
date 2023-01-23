# 常用命令
```
ufw version			  #查看版本信息
ufw enable            #启用防火墙
ufw disable           #禁用防火墙
ufw reload            #重载防火墙
ufw reset             #重新设置防火墙 (注意：这将禁用UFW并删除之前定义的任何规则)
ufw verbose           #查看防火墙策略
```
使用
最新版的UFW默认启用了IPV6配置，你也可以通过以下命令进行检查，可以看到以下输出
```shell
root@cyy:~# cat /etc/default/ufw | grep -i ipv6
# Set to yes to apply rules to support IPv6 (no means only IPv6 on loopback
IPV6=yes
```
如果输出为IPV6=no，可以使用vim编辑该文件将其改为yes。

一、设置UFW默认策略
默认情况下，UFW默认策略设置为阻止所有传入流量并允许所有传出流量，你可以使用以下命令来设置自己的默认策略：
```shell
# ufw default allow outgoing 
# ufw default deny incoming
```

# 二、设置SSH或其他规则
```shell
允许传入SSH连接，可以使用以下命令：
# ufw allow ssh
```
这将创建防火墙规则-允许端口22上的所有连接
也可以通过直接指定端口来创建等效规则
```shell
# ufw allow 22
```
# 三、设置其他规则
可以基于TCP或UDP协议来过滤数据包，命令如下：
```shell
# ufw allow 80/tcp
# ufw allow 21/udp
```
ufw status verbose #查看防火墙策略
```shell
# ufw status verbose
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), deny (routed)
New profiles: skip
To                         Action      From
80/tcp                     ALLOW IN    Anywhere
21/udp                     ALLOW IN    Anywhere
80/tcp (v6)                ALLOW IN    Anywhere (v6)
21/udp (v6)                ALLOW IN    Anywhere (v6)
```
还可以使用以下命令随时拒绝指定端口任何传入和传出的流量：
```shell
# ufw deny 80
# ufw deny 21
```
如果要删除HTTP允许的规则，只需在原始规则前加上delete即可，如下所示：
```shell
# ufw delete allow http
# ufw delete deny 21
```
也可以按编号删除规则，使用以下命令查看规则及其编号：
```shell
# ufw status numbered
OutputStatus: active
     To                         Action      From
     --                         ------      ----
[ 1] 22/tcp                     ALLOW IN    Anywhere
[ 2] 443/tcp                    ALLOW IN    Anywhere
[ 3] 22/tcp (v6)                ALLOW IN    Anywhere (v6)
[ 4] 443/tcp (v6)               ALLOW IN    Anywhere (v6)
```
举个例子，要删除允许端口443上的HTTPS连接规则，也就是编号2（注意删除完规则后编号会更新）：
```shell
# ufw delete 2
```
# 四、特定端口范围
某些应用程序使用多个端口而不是单个端口，你可能需要使用UFW指定端口范围，指定端口范围时，必须指定规则应适用的协议（ tcp或udp ）。例如，要允许使用端口9000-9002范围内的连接，可以使用以下命令：
```shell
# ufw allow 9000:9002/tcp
# ufw allow 9000:9002/udp
```
# 五、特定的ip地址
出于某些情况，你可能需要允许/禁用来自特定IP地址的连接，如下：
```shell
# ufw allow from 192.168.29.36
# ufw deny from 192.168.29.36
```
还可以在UFW中允许IP地址范围,以下命令将允许从192.168.1.1到192.168.1.254的所有连接：
```shell
# ufw allow from 192.168.1.0/24
```
要允许IP地址192.168.29.36连接特定的端口80，可以运行以下命令：
```shell
# ufw allow from 192.168.29.36 to any port 80
```
进一步的，可以指定TCP/UDP：
```shell
# ufw allow from 192.168.29.36 to any port 80 proto tcp
```
来个复杂点的例子，
```shell
# ufw deny from 192.168.0.4 to any port 22 
# ufw deny from 192.168.0.10 to any port 22 
# ufw allow from 192.168.0.0/24 to any port 22
```
以上命令将会阻止从192.168.0.4和192.168.0.10访问端口22，但允许所有其他IP访问端口22。
以上大致简述了UFW的基本使用，日常使用基本够用。除此之外，UFW还支持更多的高级特性例如允许与特定网络接口的连接、配置NAT等等，你可以通过查看帮助文档或者搜索引擎了解其他高级命令的使用。
```
拓展：
学习以下命令用法
 logging LEVEL                 #设置日志记录为LEVEL
 allow ARGS                    #添加允许规则
 deny ARGS                     #添加否定规则
 reject ARGS                   #添加拒绝规则
 limit ARGS                    #添加限制规则
 delete RULE|NUM                 #删除规则
 insert NUM RULE                 #在NUM处插入规则
 route RULE                      #添加路由规则
 route delete RULE|NUM           #删除路由规则
 route insert NUM RULE           #在NUM插入路由规则

 status                          #显示防火墙状态
 status numbered                 #显示防火墙状态的编号列表的规则
 
 show ARG                        #报告显示防火墙
 version                         #显示版本信息

应用程序配置文件命令:
 app list                       #列表应用程序配置文件
 app info PROFILE               #显示PROFILE信息
 app update PROFILE             #更新配置文件
 app default ARG                #设置默认应用策略 
```

# 转自
[ubuntu防火墙，ufw命令](https://blog.csdn.net/qq_54947566/article/details/124426713)