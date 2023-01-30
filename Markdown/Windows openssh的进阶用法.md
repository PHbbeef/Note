# Windows openssh的进阶用法
在C:\Users\admin\.ssh\新建一个"config"
## 配置(不同的使用)
```
Host root
	HostName www.baidu.com
	User root
	IdentityFile ~/.ssh/root
	
Host 乌班图
	HostName 114.114.114.114
	User ubuntu
	IdentityFile C:\Users\admin\.ssh\id_rsa
```
```sh
Host            #快捷方式名
HostName        #远程服务器连接域名或IP
User            #账户
IdentityFile    #密钥位置
```

### 连接到远程服务器
打开cmd
```
ssh 乌班图
```
或
```
ssh root
```

