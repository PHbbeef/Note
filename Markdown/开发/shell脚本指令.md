# 简单指令
```bash
# 定义变量
variable=value
variable='value'   #单引号' '包围变量的值时，单引号里面是什么就输出什么
variable="value"   #以双引号" "包围变量的值时，输出时会先解析里面的变量和命令

#使用变量
author="小白"
echo $author
echo ${author}  #推荐使用这个

#命令结果赋值给变量
log=$(cat log.txt)  #第一种
log=`cat log.txt`   #第二种
```
# 自动备份文件推到git
```bash
Windows用户请借助 定时任务 实现自动执行脚本文件 参考文档

Linux用户请借助 Crontab 命令实现自动执行脚本文件 参考文档
```
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