# 基本
镜像可以看做是一个类，镜像类不断实例化一个镜像可以有多个容器。

```bash
#查看docker信息
docker info
```

## 镜像

```bash
# 查看镜像
docker images

# 删除镜像
docker rmi -f [镜像ID]
```

## 容器

```bash
# 查看正在运行的容器
docker ps-
docker ps -aq     #列出所有容器ID   

# 开始停止容器
docker start [容器ID]
docker stop [容器ID]

# 删除容器
docker rm -f [容器ID]
docker rm -f $(docker ps -aq)     #删除所有容器

# 进入容器
docker exec -it [容器ID] [指定目录]


#查看容器日志
docker logs [容器ID]
```


# 其他

## 拉取 Huginn

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
