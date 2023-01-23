# 安卓手机用qemu运行openwrt
转载自 [安卓手机用qemu运行openwrt](https://www.bilibili.com/read/cv14075672)
## 准备
### 替换源
替换国内清华源参考 https://mirrors.tuna.tsinghua.edu.cn/help/termux/
### 安装安装openssh

    pkg upgrade
    pkg install openssh
    sshd
可以再输入一条 echo "sshd" >> ~/.bashrc  这样不用每次打开终端都要运行一次sshd 需要注意的是，我们平常开启的ssh服务端口是22，但是Termux开启的ssh服务端口是在8022 接着我们输入 ifconfig 查看自己的ip地址（若是需要连接电脑和手机需要在同一个的WiFi下）

## 开始安装
	#跟新源
    apt-get update
	
	#查看qemu有什么架构包
	apt search qemu

    #安装qemu工具包
    apt install qemu-utils
    
## 选择架构
当然，我的是arm64位架构的，模拟arm的性能损耗会更低

    #安装模拟aarch64架构的
    apt install qemu-system-aarch64-headless

    #安装x86_64架构的
    apt install qemu-system-x86-64-headless

### 进行arm架构的教程   
这里先采用官方releases版固件，你们成功后可以换其他充满插件的固件、或者手动构建armvirt镜像，替换执行命令里固件名字就行了

    #下载arm固件
    wget https://mirrors.tuna.tsinghua.edu.cn/openwrt/releases/19.07.8/targets/armvirt/64/openwrt-19.07.8-armvirt-64-root.ext4.gz

    #解压（与X86会有combined不同的是，还有指定kernel，所以还要内核镜像。你们使用自己编译的固件时，记得也把内核文件复制到手机）
    gzip -d openwrt-19.07.8-armvirt-64-root.ext4.gz
    
    wget https://mirrors.tuna.tsinghua.edu.cn/openwrt/releases/19.07.8/targets/armvirt/64/openwrt-19.07.8-armvirt-64-Image
这个时候输入查看命令ls执行可以看到文件openwrt-19.07.8-armvirt-64-root.ext4 和 openwrt-19.07.8-armvirt-64-Image

### 进行X86架构的教程

    #下载
    wget https://mirrors.tuna.tsinghua.edu.cn/openwrt/releases/19.07.8/targets/x86/64/openwrt-19.07.8-x86-64-combined-ext4.img.gz

    #解压
     gzip -d openwrt-19.07.8-x86-64-combined-ext4.img.gz

## 开始运行openwrt

    #arm架构
    qemu-system-aarch64 -M virt -m 1024m -kernel openwrt-19.07.8-armvirt-64-Image -append "root=fe00" -hda openwrt-19.07.8-armvirt-64-root.ext4 -no-reboot -nographic -cpu cortex-a53 -smp 4 -net nic -net user,id=wan,hostfwd=tcp::7080-:80,hostfwd=tcp::7022-:22

    #X86架构
    qemu-system-x86_64 -m 1024m -hda openwrt-19.07.8-x86-64-combined-ext4.img -no-reboot -nographic  -smp 2 -net nic -net user,id=wan,hostfwd=tcp::7080-:80,hostfwd=tcp::7022-:22
    

	
命令说明：

-M 是模拟的机器，可以执行qemu-system-aarch64 -M help查看列表，可以看到有树莓派的，所以也可以直接用树莓派的固件

-m 是分配内存大小 我这里分配1024mb

-kernel是指定内核

-append cmdline 设置Linux内核命令行和启动参数

我这里”root=fe00“指定根的块设备是fe00，如果你没有指定这个，内核将列出可用的块设备并重新启动，之后你们自己的固件可以取消这个”root=fe00“看可用块设备列表，再修改填上。我之前在这徘徊了很久啊。

-hda是指定硬盘镜像

--no-reboot 就是字面意思，里面客户系统如果重启就会直接退出qemu，重启相当于关机退出qemu。可以不要这条，这样客户系统可以进行重启的操作

-nographic 关闭qemu的图形化界面输出。也可以去掉，然后加上--vnc :1   以vnc为图像模式输出到”显示器”，并占用vnc 1端口，vnc访问手机ip:5901显示进入图像界面。-nographic与--vnc不同的是执行运行后不会立即有回显。

-cpu cortex-a53 模拟cortex-a53类型的处理器，因为前面查询我的骁龙625是cortex-a53类型处理器，模拟这个性能损失较小。可以输入qemu-system-aarch64 -cpu help查看可模拟列表

-smp 核数，给cpu分配核数

-net nic  就是快速配置网卡。后面net user,id=wan,hostfwd=tcp::7080-:80,hostfwd=tcp::7022-:22配置网卡网络模式为用户模式（nat模式，使用主机网络nat联网），分配id标识为wan，hostfwd是端口重定向参数，可以加逗号多个使用，很清晰可以自己根据需要增加和删改，这里我把主机7080端口重定向到客户系统的80端口，把主机7022端口重定向到客户系统的22端口。这两个端口分别是web网页管理地址端口、ssh端口。还有具体设置网卡可以百度搜索qemu网络模式。

执行后耐心等待跑码

出现第一个框就可以按回车输命令了，但是不急，等第二个

出现br-lan: port 1(eth0) entered forwarding state就是启动好了（改过下面以后，第二次运行是出现8021q: adding VLAN 0 to HW filter on device eth0）

## 修改
因为第一个网卡默认是分配给lan的，所以我们要改一下，分给wan

    vi /etc/config/network

    #按键盘i就可以编辑进行增删了，把config interface 'wan' 那整部分改成以下这样（保存退出）
    config interface 'wan'
        option ifname 'eth0'
        option proto 'dhcp' 

    #重启网络服务
    /etc/init.d/network restart

    #ping外网
    ping www.baidu.com

    #开放wan的80端口
    iptables -I INPUT -p tcp --dport 80 -j ACCEPT
前面执行命令的时候，我们已经弄好手机7080端口重定向客户机的80端口了，所以我们在浏览器访问手机ip加7080端口，就可以进入openwrt的管理页面啦！

连上openwrt的ssh，前面执行命令的时候，我们已经弄好手机7022端口重定向客户机的22端口了，只要在openwrt开放22端口，就可以手机ip加7022连上了

    iptables -I INPUT -p tcp --dport 80 -j ACCEPT

你可以根据需要设置好重定向端口和开放端口，例子很明白了

