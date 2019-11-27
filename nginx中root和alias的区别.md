# nginx中root和alias的区别

mkdir  -p   /www/abc

echo  "/www"  >  index.html

[root@localhost www]# ls
abc  index.html

echo  "/www/abc"  >  abc/index.html

vim   /usr/local/nginx/config/nginx.conf

location /abc {
            root  /www;
            index index.html index.htm;
        }

访问出来的站点根目录在 /www/abc

echo "/www/abc" > index.html

 vim /usr/local/nginx/conf/nginx.conf

location /abc {
            alias  /www;
            index index.html index.htm;
        }

访问出来的站点根目录在 /www

## nginx的虚拟主机

vim  /usr/local/nginx/conf/nginx.conf

34     server {
 35         listen   192.168.2.20;
 36         server_name www.kgc.com;
 37         location / {
 38                 root /www/kgc;
 39                 index index.html;
 40                 }
 41         }
 42     server {
 43         listen       192.168.2.10;

ifconfig ens33:0 192.168.2.20

 mkdir /www/kgc

echo "/www/kgc" > /www/kgc/index.html

[root@localhost abc]# cat /www/kgc/index.html 
/www/kgc

 killall nginx

 nginx

[root@localhost abc]# curl 192.168.2.20
/www/kgc

[root@localhost abc]# curl 192.168.2.10
aa

## 基于域名的虚拟主机

vim   /etc/hosts

192.168.10.1  www.kgc.com
192.168.10.1  www.abc.com

 vim   /usr/local/nginx/conf/nginx.conf

 server {
        listen    192.168.10.1;
        server_name  www.kgc.com;
        location / {
                root  /www/kgc;
                index  index.html;
        }
        }
    server {
        listen     192.168.10.1;
        server_name  www.abc.com;

killall  nginx

 nginx

curl  www.kgc.com

/www/kgc
[root@localhost ~]# curl  www.abc.com
aa

## 基于端口的虚拟主机

vim   /usr/local/nginx/conf/nginx.conf

 server {
        listen    192.168.10.1:80;
        server_name  www.kgc.com;
        location / {
                root  /www/kgc;
                index  index.html;
        }
        }
    server {
        listen     192.168.10.1:8080;
        server_name  www.abc.com;

[root@localhost ~]# killall nginx
[root@localhost ~]# nginx
[root@localhost ~]# curl  192.168.10.1:80
/www/kgc
[root@localhost ~]# curl  192.168.10.1:8080
aa

## 修改默认页面

vim  /usr/local/nginx/conf/nginx.conf

把端口去掉

 server {
        listen    192.168.10.1;
        server_name  www.kgc.com;
        location / {
                root  /www/kgc;
                index  index.html;
        }
        }
    server {
        listen     192.168.10.1;
        server_name  www.abc.com;

查看

[root@localhost www]# curl 192.168.10.1
/www/kgc

更改

 vim  /usr/local/nginx/conf/nginx.conf

  server {
        listen    192.168.10.1;
        server_name  www.kgc.com;
        location / {
                root  /www/kgc;
                index  index.html;
        }
        }
    server {
        listen     192.168.10.1 default;
        server_name  www.abc.com ;

killall  nginx

nginx  

curl  192.168.10.1

[root@localhost www]# curl 192.168.10.1
aa

## 地址重写

vim  /usr/local/nginx/conf/nignx.conf

 server {
        listen    192.168.10.1;
        server_name  www.kgc.com;
        location / {
                root  /www/kgc;
                index  index.html;
                if ($host != 'abc'){
                        rewrite ^/(.*)$ http://www.abc.com/$1 permanent;
        }
        }
        }
    server {
        listen     192.168.10.1;
        server_name  www.abc.com ;

killall  nginx

nginx

firefox  http://www.kgc.com



## 基于用户访问控制

auth_basic   用来支持http的基本认证（用户和密码的验证）

vim   /usr/local/nginx/conf/nginx.conf

  location / {
            root   html;
            index  aa.html index.html index.htm;
            auth_basic "welcome VIP";
            auth_basic_user_file /usr/local/nginx/conf/.htpasswd;
        }

yum provides htpassswd

yum -y install  httpd-tools

htpasswd  -c  /usr/local/nginx/conf/.htpasswd haha

123.com

123.com

killall  nginx

nginx

firefox  http://www.abc.com

## nginx隐藏版本号

vim  /usr/lcoal/nginx/conf/nginx.conf

http {
    include       mime.types;
    default_type  application/octet-stream;
    server_tokens off;

killall  nginx

nginx

cuel -l www.kgc.com

HTTP/1.1 301 Moved Permanently
Server: nginx
Date: Wed, 27 Nov 2019 03:05:55 GMT
Content-Type: text/html
Content-Length: 178
Connection: keep-alive
Location: http://www.abc.com/

## nginx的访问控制（只允许本机访问，不允许其他主机访问）

vim  /usr/lcoal/nginc/conf/nginx.conf

 server {
        listen    192.168.10.1;
        server_name  www.kgc.com;
        location / {
                root  /www/kgc;
                index index.html
                allow  192.168.10.1
                deny all;
        }
        }

killall  nginx

nignx

curl  www.kgc.com