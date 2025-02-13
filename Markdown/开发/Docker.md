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



## 拉取 Huginn

```bash
# 安装docker
curl -fsSL https://test.docker.com -o test-docker.sh
sudo sh test-docker.sh
#如果要指定拉取仓库修改 vim /etc/docker/daemon.json
```

```bash
#开启 docker
systemctl start docker

#查看状态
systemctl status docker

#开机自启动
systemctl enable docker
```

```bash
#从仓库拉取镜像，创建并指定端口创建启动一个容器
docker run -it -p 3000:3000 huginn/huginn

#查看Docker中现有的镜像
docker image ls

#删除镜像
# 注！惊喜删除还有容器要删除
docker image rm huginn/huginn

#使用本地镜像运行容器：在确认镜像名称和标签后，你可以使用类似下面的命令来运行容器
docker run -it -p 3000:3000 huginn/huginn:latest

#查看运行中的容器
docker ps

#停止容器
docker stop <容器ID或名称>
```

## 设置代理
  由于国内网络环境代理是个不错的选择

`docker pull`命令是由 dockerd 守护进程执行。而 dockerd 守护进程是由 systemd 管理。因此，如果需要在执行`docker pull` 命令时使用 HTTP/HTTPS 代理，需要通过 systemd 配置。

```bash
#为 dockerd 创建配置文件夹
mkdir -p /etc/systemd/system/docker.service.d

# 为 dockerd 创建 HTTP/HTTPS 网络代理的配置文件，文件路径是 /etc/systemd/system/docker.service.d/http-proxy.conf 。并在该文件中添加相关环境变量。
[Service]
Environment="HTTP_PROXY=http://proxy.example.com:8080/"
Environment="HTTPS_PROXY=http://proxy.example.com:8080/"
Environment="NO_PROXY=localhost,127.0.0.1,.example.com"
```

为 docker 容器设置网络代理
```bash
# 在容器运行阶段，如果需要使用 HTTP/HTTPS 代理，可以通过更改 docker 客户端配置，或者指定环境变量的方式。
# 更改 docker 客户端配置：创建或更改 ~/.docker/config.json，并在该文件中添加相关配置。
{
 "proxies":
 {
   "default":
   {
     "httpProxy": "http://proxy.example.com:8080/",
     "httpsProxy": "http://proxy.example.com:8080/",
     "noProxy": "localhost,127.0.0.1,.example.com"
   }
 }
}
```

为 docker build 过程设置网络代理
```bash
# 在容器构建阶段，如果需要使用 HTTP/HTTPS 代理，可以通过指定 "docker build" 的环境变量，或者在 Dockerfile 中指定环境变量的方式。
docker build \
    --build-arg "HTTP_PROXY=http://proxy.example.com:8080/" \
    --build-arg "HTTPS_PROXY=http://proxy.example.com:8080/" \
    --build-arg "NO_PROXY=localhost,127.0.0.1,.example.com" .
```
