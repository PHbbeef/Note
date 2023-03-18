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

### 编译和安装
```shell
$ make && make install
```

