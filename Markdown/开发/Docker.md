# 基本
镜像可以看做是一个类，镜像类不断实例化一个镜像可以有多个容器。

```bash
#查看docker信息
docker info

#指定 huginn 开机自启
docker update --restart=always huginn
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
docker ps
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

## 数据库

### MySQL

```bash
# 修改绑定的IP为 0.0.0.0
#这是为了让docker能连接到本地的数据库。
#默认的 127.0.0.1 只能从本地访问。
vim /etc/mysql/mysql.conf.d/mysqld.cnf
```

```bash
# 登录 MySQL 数据库
mysql -u root -p

 #新建 huginn 用户密码为 PASSWORD
create user 'huginn'@'%' identified by 'PASSWORD';
#这个是允许从任意地方登录。
#如果要限制从docker里启动，先ifconfig看下 docker0 的IP，一般是172.17.0.1，则 docker IP 为172.17.0.*，于是
create user 'huginn'@'172.17.0.%' identified by 'PASSWORD';

# 赋予权限
GRANT ALL PRIVILEGES ON *.* TO 'huginn'@'172.17.0.%' IDENTIFIED BY 'PASSWORD' WITH GRANT OPTION;
flush privileges;
```

```bash
docker run  --name huginn \
    -p 3000:3000 \
    --restart always \
    -e MYSQL_PORT_3306_TCP_ADDR=172.17.0.1 \
    -e HUGINN_DATABASE_NAME=huginn \
    -e HUGINN_DATABASE_USERNAME=huginn \
    -e HUGINN_DATABASE_PASSWORD=PASSWORD \
    huginn/huginn
```

### MariaDB

```bash
# /etc/mysql/mariadb.cnf包含了下面的所以.cnf文件；配置文件目录
!includedir /etc/mysql/conf.d/
!includedir /etc/mysql/mariadb.conf.d/
```