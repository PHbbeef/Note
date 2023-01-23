# Linux swap
```sh
#显示当前分区情况
# free -m
```

```sh
#关闭分区
# swapoff -a
```

创建要作为 Swap 分区文件（其中 /var/swapfile 是文件位置，bs*count 是文件大下，例如以下命令就会创建一个 4G 的文件）：
```sh
# dd if=/dev/zero of=/var/swapfile bs=1M count=4096
```

建立 Swap 的文件系统（格式化为 Swap 分区文件）：
```sh
# mkswap /var/swapfile
```

启用 Swap 分区：
```sh
# swapon /var/swapfile
```

设置开启启动，在 /etc/fstab 文件中加入一行代码：
```sh
# /var/swapfile swap swap defaults 0 0
```