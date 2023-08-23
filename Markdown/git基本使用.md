## 基本指令
```bash
# 还原所有修改，不会删除新增的文件
git checkout .
```

## 撤销
```bash
git revert HEAD   # 撤销前一次操作   
git revert HEAD~  # 撤销前前一次操作   
git revert commit # 撤销指定操作   
```

## 回退提交
注：回退提交并不会把上次内容删除
```shell
#回退到上次版本
git reset --hard HEAD^

#回退到指定
git reset --hard <hash>
```

## 克隆远程指定分支
```
git clone -b https://github.com***
```

### 前仓库的一个分支push到另一个仓库的指定分支（注：提交的记录会一块上传）
假设有2个仓库uu和uunew
* uu: 当前仓库         
* uunew: 目标仓库

我们想把当前仓库【uu】的指定分支【R1_2212-flw】推给另一个仓库【uunew】的指定分支【R1_2212】
```
git pull          # 在当前仓库操作：更新代码库
```
```
git remote    # 查看当前仓库origin 只有一个，接下来我们要add 另一个仓库的origin
```

```
git remote add uunew R1_2212  # uunew:远程仓库名称，可以随便起个方便记忆的， 目的是在本地添加一个新的远程连接，添加一个 R1_2212分支
```
uunew后面是uunew的一个分支，可以指定为master或你要push的目标分支，都可以。  执行完命令后可以再次通过 git remote 查看现在的有几个origin，大家可以自行试一试

```
## 这里是新加个远程连接 设置上目标仓库的url地址
git remote set-url uunew https://gitee.com/greatoak/uunew.git
```

```
#最好是先切换到要push的当前分支上，然后再push
git checkout R1_2212-flw             # 切换到当前分支R1_2212-flw
git push uunew R1_2212-flw:R1_2212   # uunew是另一个仓库的orgin 含义是把当前分支R1_2212-flw 推到 目标分支R1_2212上
```

### 转载 [GIT操作：把当前仓库的一个分支push到另一个仓库的指定分支](https://blog.csdn.net/gct/article/details/128415329?spm=1001.2101.3001.6650.1&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1-128415329-blog-122924844.235%5Ev32%5Epc_relevant_increate_t0_download_v2&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1-128415329-blog-122924844.235%5Ev32%5Epc_relevant_increate_t0_download_v2&utm_relevant_index=2)

## git commit提示设置
```
git config --global commit.template /home/zqchen/.zqchen_gitmessage.txt
```