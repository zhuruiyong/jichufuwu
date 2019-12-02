# nginx的反向代理和负载均衡

192.168.2.10  nginx  这个是做反向代理的

192.168.2.20  nginx  （nginx  httpd）

192.168.2.30  nginx   （nginx  httpd）

## nginx反向代理服务器

代理服务器：

正向代理：传统代理  透明代理（架构一）

反向代理：（nginx） （CDN）    是把一些静态的资源缓存在服务器上，当用户有强求的时候，就直接而返回反向代理上面的资源给用户，而反向代理资源上，如果没有请求的资源就去后端进行获取，获取完之后在返回给客户端，并缓存在自己上面（静态资源）

## 简单的反向代理

反向代理端

vim /usr/local/nginx/conf/nginx.conf

IP写web服务器

 location / {
            #root   html;
            proxy_pass  http://192.168.10.1:80;
            index  index.html index.htm;
        }

firefox  http://192.168.10.3

## 负载均衡：

为了保证web服务器的高可用，不需要储存资源，只是需要转发请求。 

nginx的负载均衡是基于反向代理之上的

vim  /usr/local/nginx/conf/nginx.conf

 #gzip  on;
     upstream web {
        server  192.168.10.1:80;
        server  192.168.10.2:80;
}

location / {
            #root   html;
            proxy_pass  http://web;
            index  index.html index.htm;
        }

在php上

cd /www/

mv  index.php forum.php

反向代理访问

firefox http：//192.168.10.3

![](D:\github\jichufuwu\7.gif)

