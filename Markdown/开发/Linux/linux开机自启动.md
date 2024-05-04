

# **systemd**

创建文件

```shell
vim /etc/systemd/system/qbittorrent.service
```

基本配置

```Shell
[Unit]
Description=qBittorrent Daemon Service
After=network.target
[Service]
User=root
ExecStart=/usr/bin/qbittorrent-nox
ExecStop=/usr/bin/killall -w qbittorrent-nox
[Install]
WantedBy=multi-user.target
```

## Systemd文件结构

`[Unit]`：服务的全局信息和依赖性声明，如服务名称、描述等。

`[Service]`：指定服务的具体配置，如服务执行的命令、工作目录等。

`[Install]`：指定服务的安装方式，如服务的启动级别等。


###  [Unit]配置
`Description`：对服务的简短描述。

`Before`：定义服务在其他服务之前启动。

`After`：定义服务在其他服务之后启动。

`Requires`：定义服务启动需要哪些其他服务已启动，否则无法启动。

`PartOf`：定义该服务是其他服务的一部分，如果其他服务停止，该服务也会停止。

`Wants`：定义服务启动时可同时启动哪些其他服务。

`Condition...`：定义服务启动的条件，如ConditionPathExists表示某个路径存在时才启动该服务。


### [Service]配置
`Type`：服务类型，可以是simple、forking、ondemand、notify等。

`ExecStart`：服务启动命令，可以是单个命令、脚本文件、或者多个命令组成的脚本。

`ExecStop`：停止服务的命令。

`User`：定义服务运行的用户。

`Group`：定义服务运行的用户组。

`PrivateTmp`：将服务的/tmp目录挂载到私有的命名空间中，以增强安全性。

`Restart`：定义服务异常退出时如何重启。

`WorkingDirectory`：定义服务工作目录。

`Environment`：定义服务的环境变量等。

`ProtectSystem`：防止服务对系统文件进行修改。

`NoNewPrivileges`： 防止服务通过setuid或setgid等提升权限。


### [Install]配置
`WantedBy`：定义在哪些系统运行级别下启用此服务。

`RequiredBy`：启动其他系统服务时必需启动此服务。