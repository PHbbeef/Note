## 生成密钥
```shell
# ssh-keygen -t rsa -C "tconcave9@gmail.com"
```
其中，tconcave9@gmail.com为你的电子邮箱，不需要制定邮箱可以简化为：
```shell
# ssh-keygen
```

其中：
* <font color="red">Enter file in which to save the key</font>：密钥存放地址，默认为当前用户目录下的.ssh文件夹下
* <font color="red">Enter passphrase</font>：保护私钥的密码，一般留空，直接和上面一样回车即可
* <font color="red">Enter same passphrase again</font>：确认私钥密码（不解释了……）
* 


## 配置说明
生成后，进入用户名文件夹即可看到我们生成的密钥：
* <font color="red">id_rsa</font>：生成的私钥，保留在电脑即可
* <font color="red">id_rsa.pub</font>：生成的公钥，打开后，复制内容，后文部署到服务器上
* 之后，进入<font color ="red">.ssh</font>文件夹内（如果没有就使用mkdir命令创建）,并使用vim创建并编辑<font color="red">authorized_keys</font>文件：
* 粘贴公钥即,保存并退出即可

## 相关设置
密钥登录相关设置
```shell
# 修改它们的权限，防止其他人读取
chmod 600 ~/.ssh/id_rsa
chmod 600 ~/.ssh/id_rsa.pub

# authorized_keys文件的权限要设为644，即只有文件所有者才能写。如果权限设置不对，SSH 服务器可能会拒绝读取该文件
chmod 644 ~/.ssh/authorized_keys

# 禁止root登录
PermitRootLogin no

# 开启密钥登录
PubkeyAuthentication yes

# 关闭密码登录
PasswordAuthentication no

```

## 相关问题
```shell
# 1.禁止密码登录失效
/etc/ssh/sshd_config.d
```