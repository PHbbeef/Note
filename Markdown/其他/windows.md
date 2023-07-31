# Windows 系统开启系统自带的webdav挂载功能
接下来是系统自带的方法，需要执行这三个内容
+ 修改注册表内容
+ 启动webclient服务
+ 添加网络映射
## 修改注册表内容
```
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\WebClient\Parameters
```
需要修改 BasicAuthLevel 为2，FileSizeLimitInBytes 为最大值

代码部分
```
Windows Registry Editor Version 5.00
[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\WebClient\Parameters]
"AcceptOfficeAndTahoeServers"=dword:00000001
"BasicAuthLevel"=dword:00000002
"ClientDebug"=dword:00000000
"FileAttributesLimitInBytes"=dword:000f4240
"FileSizeLimitInBytes"=dword:ffffffff
"InternetServerTimeoutInSec"=dword:0000001e
"LocalServerTimeoutInSec"=dword:0000000f
"SendReceiveTimeoutInSec"=dword:0000003c
"ServerNotFoundCacheLifeTimeInSec"=dword:0000003c
"ServiceDebug"=dword:00000000
"ServiceDll"=hex(2):25,00,53,00,79,00,73,00,74,00,65,00,6d,00,52,00,6f,00,6f,\
  00,74,00,25,00,5c,00,53,00,79,00,73,00,74,00,65,00,6d,00,33,00,32,00,5c,00,\
  77,00,65,00,62,00,63,00,6c,00,6e,00,74,00,2e,00,64,00,6c,00,6c,00,00,00
"ServiceDllUnloadOnStop"=dword:00000001
"SupportLocking"=dword:00000001
```
## 启动 webclient服务
用管理员身份运行 CMD ，依次执行
```
net stop webclient
```
```
net start webclient
```
## 添加网络映射部分
直接在桌面，此电脑，右键，映射网络驱动器
勾选上 使用其他凭据登录