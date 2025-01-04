### Windos挂载ext4

安装WSL
```
wsl install
```

获取驱动器列表
```shell
wmic diskdrive list brief
```

挂载指定硬盘
```shell
wsl --mount \\.\PHYSICALDRIVE1 --partition 1

Caption: 磁盘驱动器的简短描述，通常与 Model 类似，但可能包含更多的用户友好信息。
DeviceID: 磁盘驱动器的设备标识符。
Model: 磁盘驱动器的型号。
Partitions: 磁盘驱动器上的分区数量。
Size: 磁盘驱动器的大小，以字节为单位。




要挂载指定文件系统，可使用以下命令：
wsl --mount \\.\PHYSICALDRIVE1 --partition 1 -t ext4
```

问题
```
默认挂载点为/mnt/wsl

sudo chmod -R 777 /mnt/wsl/PHYSICALDRIVE5p2
```

### Linux磁盘备份
```
if: 是指定输入文件或设备
of: 是指定输出文件或设备
bs: 是指定数据块的大小
count: 是指定要复制的数据块数量

# 备份磁盘到文件
dd if=/dev/sda of=/backupdir/mirror.img bs=4M

# 恢复备份文件到新磁盘
dd if=/backupdir/mirror.img of=/dev/sdb bs=4M

```