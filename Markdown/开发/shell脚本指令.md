# 自动备份文件推到git
```bash
#!/bin/bash
git status
echo "####### 开始自动Git #######"
current_time=$(date "+%Y/%m/%d -%H:%M:%S") # 获取当前时间
echo ${current_time} # 显示当前时间
git add .
git commit -m "modified ${current_time}" # 远程仓库可以看到是什么时间修改的...
git push origin main
echo "####### 自动Git完成 #######"
sleep 10s
```

# 判断文件是否存在
```bash
#shell中有-d,-f,-e,-n每个判断的作用不一样
```
```bash
 if [ ! -f "./test.sh" ];then
  echo "文件不存在"
  else
  echo "文件存在"
 fi
```