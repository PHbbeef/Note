# 基本指令
```bash
# 查看镜像
docker images

# 运行镜像
docker run <hello-docker>

# 构建镜像
镜像名，当前目录
docker build -t hello-docker .
```


# 其他
```bash
FROM node:14-alpine
COPY index.js /index.js
CMD node /index.js
```
## 每一行指令说明：
镜像是基于层次结构构建的每一层都基于上一层；node这个镜像已经是基于Linux来构建的可以拿来直接使用
+ 使用node14版本镜像，alpine是轻量的linux发行版
+ 两个参数(源路径，目标路径),源路径：相当于Dockerfile文件的路径；目标路径：镜像的路径
+ 两个参数(可执行程序的名字，可执行程序接受的参数) 
