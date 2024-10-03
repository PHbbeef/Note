### 使用模拟器
这里以雷电模拟器为例

打开雷电模拟器安装目录（可以通过任务管理器查看），在模拟器的目录下会有ADB程序。

```shell
# 默认连接到手机,默认端口为5037(USB)
adb devices 	

# 手机IP地址(WiFi)
adb connect/disconnect 	

# PC13000端口转发到15000
adb forward tcp:13000 tcp:15000	

# 指定安卓ADB监听设备5555
adb tcpip 5555	


# Visual Studio不显示模拟器设备
# 先使用 adb devices 连接设备，在打开Visual Studio

```

