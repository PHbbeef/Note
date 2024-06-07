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

