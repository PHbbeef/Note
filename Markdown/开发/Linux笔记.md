```
#配合管道符筛选指定名
dpkg -l |grep vim
```

# 环境变量
Linux是一个多用户操作系统，每个用户都有自己专有的运行环境。用户所使用的环境由一系列变量所定义，这些变量被称为环境变量。系统环境变量通常都是大写的。

在终端运行可执行程序时是首先会在当前目录查找，如果找不到则会在环境变量中去找。
+ 环境变量告诉可执行程序的绝对路径，以便可以不在当前目录执行程序。

## 常用指令
```shell
#添加环境变量值
export DOTNET_ROOT=$HOME/dotnet
PATH=$PATH:$HOME/dotnet

#删除环境变量（删除PATH变量）
unset PATH

#查看PATH变量的值（或路径）
echo $PATH

#查看打印命令
 printenv
 env
```


# 加入sudo用户组
```bash
#切换root用户
su -

usermod -aG sudo [要加入的用户名]
```
