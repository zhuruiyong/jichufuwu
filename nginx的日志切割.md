# nginx的日志切割

vim  /usr/local/nginx/conf/nginx.conf

#log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

#access_log  logs/access.log  main;

#log_format	定义日志格式的关键字

main	日志格式的名字

$remote_addr	日志中记录客户端的ip

$remote_user	日志中记录客户端访问的用户

[$time_local]	在日志中记录访问的时间

$request		用户的请求方式和请求的资源

'$status $body_bytes_sent "$http_referer" '	web访问的信息

'"$http_user_agent" 		客户端使用浏览器的信息

"$http_x_forwarded_for"		记录请求包经过而客户端和代理服务器的信息



vim  /usr/local/nginx/conf/nginx.conf

添加：

log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    log_format  kgc  '$remote_addr - $remote_user [$time_local] "$request" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
server {
        listen     192.168.10.1;
        server_name  www.abc.com ;

        #charset koi8-r;
    
        access_log  logs/abc.access.log  kgc;
killall nginx

nginx

curl  www.abc.com

## 编辑脚本

vim  /nginx.sh

#!/bin/bash
d=`date -d "-1 day" +%Y%m%d`
logdir="/usr/local/nginx/logs"
nginx_pid="/usr/local/nginx/logs/nginx.pid"

cd  $logdir
for log in `ls *.log`
do
mv $log $log-$d
done
/bin/kill -HUP `cat $nginx_pid`



chmod  a+x  nginx.sh

./nginx.sh

ls  /usr/local/nginx/logs

![](D:\github\jichufuwu\1.gif)

创建计划任务

crontab -e

0 0 * * * ./nginx.sh

crontab -l

## 访问图片不记录日志

vim  /usr/local/nignx/conf/nginx.conf

       access_log  logs/abc.access.log  kgc;
    
        location / {
            root   html;
            index  aa.html index.html index.htm;
        }
        location /abc {
            alias  /www;
            index  index.html  index.htm;
        }
        location ~ \.(png|jpg|gif)$ {
                expires 7d;
                access_log off;
        }
killall nginx

nginx

 cd /usr/local/nginx/html/

vim  a.png

ls

![](D:\github\jichufuwu\2.gif)

重新开启一个终端

ctrl+shift+n

tail -f /usr/local/nginx/conf/nginx.conf/logs/abc.access.log

返回另一个终端

 curl  www.abc.com/a.png
aaaaaaaaa

curl -I www.abc.com/a.png

![](D:\github\jichufuwu\3.gif)

## 防盗链  （图片和静态一般都会做防盗链）

valid_referers	加入防盗链

none 	代表直接访问

blocked	表示被防火墙标记来路

server_names   域名访问

vim /usr/local/nginx/conf/nginx.confg

location ~ \.(png|jpg|gif)$ {
                expires 7d;
                access_log off;
                valid_referers none  blocked server_names *.abc.com;
                if ($invalid_referer){
                        return 403;
        }

killall  nginx

nginx

curl  -x  192.168.10.1:80 www.abc.com/a.png -I

HTTP/1.1 200 OK
Server: nginx
Date: Thu, 28 Nov 2019 02:44:35 GMT
Content-Type: image/png
Content-Length: 10
Last-Modified: Thu, 28 Nov 2019 01:56:07 GMT
Connection: keep-alive
ETag: "5ddf2937-a"
Expires: Thu, 05 Dec 2019 02:44:35 GMT
Cache-Control: max-age=604800
Accept-Ranges: bytes

curl -e "http://www.baidu.com" -x 192.168.10.1:80 www.abc.com/a.png -I
HTTP/1.1 403 Forbidden
Server: nginx
Date: Thu, 28 Nov 2019 02:46:59 GMT
Content-Type: text/html
Content-Length: 162
Connection: keep-alive

## https:

killall  nginx

cd /usr/src/nginx-1.11.5/

重新编译https

./configure --prefix=/usr/local/nginx --with-http_stub_status_module --group=nginx --user=nginx --with-http_ssl_module

make

cp /usr/local/nginx/sbin/nginx  /usr/local/nginx/sbin/nginx.bak

cp  ./objs/nginx /usr/local/nginx/sbin/

## 颁发证书

openssl genrsa -out /file.key 2048

openssl  req -new -key /file.key -out /file.csr -days 365

openssl x509 -req -in /file.csr -signkey /file.key -out /file.crt -days 365

vim /usr/local/nginx/conf/nginx.conf

在最下面的server去掉注释

server {
        listen       443 ssl;
        server_name  www.abc.com;

        ssl_certificate      /file.crt;
        ssl_certificate_key  /file.key;
    
        ssl_session_cache    shared:SSL:1m;
        ssl_session_timeout  5m;
    
        ssl_ciphers  HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers  on;
    
        location / {
            proxy_pass http://www.abc.com;
            index  index.html index.htm;
        }
    }
nginx

firefox https://www.abc.com

![](D:\github\jichufuwu\4.gif)vim    /usr/lcoa/nignx/conf/nginx.conf

location / {
            root  /qqq;
            index  index.html index.htm;
        }
    }

cd   /

mkdir  qqq

echo  "qqq"  >  qqq/index.html

killall  nginx

nginx

firefox  https://www.abc.com

![](D:\github\jichufuwu\5.gif)

## 隐藏程序名

cd /usr/src/nginx-1.11.5/

cd src/core/

vim nginx.h 

#define NGINX_VERSION      "8.8.8"	版本号
#define NGINX_VER          "abc/" NGINX_VERSION	程序名

#define NGINX_VAR          "abc"

cd ../http/

 vim ngx_http_header_filter_module.c   #响应头部的处理模块

在49和50行

static char ngx_http_server_string[] = "Server: abc" CRLF;
static char ngx_http_server_full_string[] = "Server: abc " NGINX_VER CRLF;

错误页面的底部页脚

 vim ngx_http_special_response.c 

22 "<hr><center>" NGINX_VER "(http://www.abc.com)</center>" CRLF

 29 "<hr><c killall nginxnter>abc</center>" CRLF

 killall nginx

cd /usr/src/nginx-1.11.5/

./configure --prefix=/usr/local/nginx --with-http_stub_status_module --group=nginx --user=nginx --with-http_ssl_module && make && make install

nginx

curl -I www.abc.com

![](D:\github\jichufuwu\6.gif)