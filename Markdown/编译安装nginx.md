# 安装Nginx
## 下载依赖环境
```shell
# 安装GCC,G++
$ apt instal g++
$ apt install gcc
# 下载并解压依赖软件包
$ wget https://buildpack.oss-cn-shanghai.aliyuncs.com/static/r6d/nginx/nginx-compile-lib/pcre-8.44.tar.gz

$ wget https://buildpack.oss-cn-shanghai.aliyuncs.com/static/r6d/nginx/nginx-compile-lib/zlib-1.2.11.tar.gz

$ wget https://buildpack.oss-cn-shanghai.aliyuncs.com/static/r6d/nginx/nginx-compile-lib/openssl-1.1.1l.tar.gz

```

## 下载Nginx解压,开始编译
```shell
# 注意环境的目录地址,根据自己的情况自用选择
$ ./configure \
--prefix=/usr/local/nginx \
--with-http_ssl_module \
--with-openssl=../openssl-1.1.1l \
--with-pcre=../pcre-8.44 \
--with-zlib=../zlib-1.2.11
```

## 编译和安装
```shell
$ make && make install
```




# Nginx目录索引文件下载

## 说明
```
官方文档
http://nginx.org/en/docs/http/ngx_http_autoindex_module.html
```
```
Syntax:    autoindex on | off;
Default:    
autoindex off;
Context:    http, server, location


# autoindex on   ； 表示开启目录索引


Syntax:    autoindex_localtime on | off;
Default:    
autoindex_localtime off;
Context:    http, server, location


# autoindex_localtime on; 显示文件为服务器的时间


Syntax:    autoindex_exact_size on | off;
Default:    
autoindex_exact_size on;
Context:    http, server, location

# autoindex_exact_size on; 显示确切bytes单位
# autoindex_exact_size off; 显示文件大概单位，是KB、MB、GB


若目录有中文，nginx.conf中添加utf8编码
charset utf-8,gbk;
```

## 配置文件
```conf
server{

    listen 8888;
    server_name localhost;
    charset utf-8,gbk;
    location / {
        # root E:\download;
        root /yuchaoit/;
        autoindex on;
        autoindex_localtime on;
        autoindex_exact_size off;
    }
}
```


# 反向代理
服务端反向代理解决跨域问题

```conf
    server {
        listen       80;    #nginx监听端口
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;
		
        location / {
            root   html;
            index  index.html index.htm;
        }
		location /Air{
			proxy_pass http://localhost:3000;   #nodejs服务端监听端口
			add_header Access-Control-Allow-Origin *;   #跨域头部，星号运行全部
            #这里`Air`是`nodesjs`路由api
		}
    }
```

# 搭建WebDav
[库文件下载地址](https://github.com/arut/nginx-dav-ext-module)
``` shell

# 添加库文件
--with-http_dav_module --add-dynamic-module=../nginx-dav-ext-module


# webdav配置

```
        location /webdav {
            # 存储路径
            root /home/;

            # 用户名 密码
            auth_basic "Restricted";
            auth_basic_user_file /etc/nginx/htpasswd;
            
            dav_methods PUT DELETE MKCOL COPY MOVE;
            dav_ext_methods PROPFIND OPTIONS;
            dav_access user:rw group:rw all:rw;
            create_full_put_path on;
}
```

