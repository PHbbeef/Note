# 安装Nginx
## 下载依赖环境
```shell
# 安装GCC,G++
$ apt instal G++
$ apt install GCC
# 下载并解压依赖软件包
$ wget https://buildpack.oss-cn-shanghai.aliyuncs.com/static/r6d/nginx/nginx-compile-lib/pcre-8.44.tar.gz

$ wget https://buildpack.oss-cn-shanghai.aliyuncs.com/static/r6d/nginx/nginx-compile-lib/zlib-1.2.11.tar.gz

$ wget https://buildpack.oss-cn-shanghai.aliyuncs.com/static/r6d/nginx/nginx-compile-lib/openssl-1.1.1l.tar.gz

```

## 下载Nginx解压,开始编译
```shell
# 注意环境的目录地址,根据自己的情况自用选择
$ ./configure \
--prefix=/usr/local/Nginx \
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
```
server{

    listen 8888;
    server_name localhost;
    charset utf-8,gbk;
    location / {

        root /yuchaoit/;
        autoindex on;
        autoindex_localtime on;
        autoindex_exact_size off;
    }
}
```

