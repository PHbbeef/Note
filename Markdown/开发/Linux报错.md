E: Sub-process /usr/bin/dpkg returned an error code (1)问题
```shell
# 往往是由于你上一次安装中断，或者是断电引起的
cd /var/lib/dpkg/
sudo mv info/ info_bak          
sudo mkdir info                
sudo apt-get update             
sudo apt-get -f install         
sudo mv info/* info_bak/        
sudo rm -rf info                
sudo mv info_bak info
```


## iptables
iptables v1.8.7 (nf_tables): unknown option “--dport“ 

+ 切换一下 iptables 的版本即可

```shell
# 运行指令切换版本
sudo update-alternatives --config iptables

# 如果还报错添加环境变量
# `/lib/x86_64-linux-gnu/xtables/` 根据实际情况更改（搜`搜xtables目录`）
export XTABLES_LIBDIR=/lib/x86_64-linux-gnu/xtables/

```


