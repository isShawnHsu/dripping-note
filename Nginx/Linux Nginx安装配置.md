# Linux [Nginx](http://nginx.org/en/download.html)安装配置

存放路径

`cd /opt/nginx/`

##### 1、下载

`wget http://nginx.org/download/nginx-1.20.2.tar.gz`

##### 2、安装依赖

`yum -y install gcc zlib zlib-devel pcre-devel openssl openssl-devel`

##### 3、解压缩

`tar -zxvf nginx-1.20.2.tar.gz`

##### 4、进入解压文件夹

`cd nginx-1.20.2/`

##### 5、执行配置（添加https模块）

`./configure --with-http_ssl_module --with-http_v2_module` 

```
-prefix=PATH：指定 nginx 的安装目录
--conf-path=PATH：指定 nginx.conf 配置文件路径
--user=NAME：nginx 工作进程的用户
--with-pcre：开启 PCRE 正则表达式的支持
--with-http_ssl_module：启动 SSL 的支持
--with-http_stub_status_module：用于监控 Nginx 的状态
--with-http-realip_module：允许改变客户端请求头中客户端 IP 地址
--with-file-aio：启用 File AIO
--add-module=PATH：添加第三方外部模块
```

##### 6、编译安装(默认安装在/usr/local/nginx)

`make && make install`

##### 7、启动与重启

###### 启动：

`/usr/local/nginx/sbin/nginx`

###### 重启：

`/usr/local/nginx/sbin/nginx -s reload`

###### 停止：

`/usr/local/nginx/sbin/nginx -s stop/quit`

##### 8、反向代理配置

###### 进入目录

`cd /usr/local/nginx/conf/`

###### 修改文件`nginx.conf`

`vi nginx.conf`

```
#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}

http {
    upstream xl {                                                                                                                                        
        server 127.0.0.1:7800;                                                                                                                           
    }                                                                                                                                                    

    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    tcp_nopush     on;

    
    keepalive_timeout  65;

    gzip  off;

    server {
        listen 80;
        listen 443 ssl;
        server_name  xx.yongx.fun;

        charset utf-8;

        ssl_certificate      /home/ssl/xx.yongx.fun_nginx/5086212_plex.yongx.fun.pem;
        ssl_certificate_key  /home/ssl/xx.yongx.fun_nginx/5086212_plex.yongx.fun.key;

        ssl_session_timeout 5m;
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_prefer_server_ciphers on;


        location / {
            if ( $scheme = http ) {
                return 301 https://$host$request_uri;
            }
            proxy_pass   http://xl;
            proxy_set_header Host $http_host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Fonwarded-For $proxy_add_x_forwarded_for;
        }
}
```

