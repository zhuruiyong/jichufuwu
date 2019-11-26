# 安装nginx

## 解压安装包

tar -zxvf  nginx-1.6.0.tar.gz -C  /usr/src/

 cd /usr/src/nginx-1.6.0/

 yum -y install pcre*  openssl*

useradd  -M   -s  /sbin/nologin   nginx

./configure --prefix=/usr/local/nginx --group=nginx --user=nginx --with-http_stub_status_module

make &&  make install

优化

ln  -s /usr/local/nginx/sbin/* /usr/local/sbin/

启动nginx

nginx

查看nginx运行和端口

netstat  -anpt | grep nginx

vim  /usr/local/nginx/conf/nginx.conf

修改nginx默认索引页

 location / {
            root   html;
            index  aa.html index.html index.htm;
        }

nginx -s reload

cd /usr/local/nginx/html/

echo  "aa"  >  aa.html

查看统计模块页面

vim  /usr/local/nginx/conf/nginx.conf

 location /status {
                stub_status on;
                access_log off;
        }

nginx  -s  reload

firefox http://127.0.0.1/status

![](D:\github\jichufuwu\image\Untitled\11.gif)

Active connections: 1 	#当前活动的链接数
server accepts handled requests	
 3 3 5 	一次处理的请求数	已经处理的请求数	总共处理的请求数
Reading: 0 Writing: 1 Waiting: 0 

## nginx的启动脚本

 cd  /etc/rc.d/init.d

vim  nginx



## 平滑升级

tar -zxvf nginx-1.11.5.tar.gz  -C /usr/src/

cd  /usr/src/nginx-1.11.5/

./configure  --prefix=/usr/local/nginx --user=nginx --group=nginx --with-http_stub_status_module && make

mv /usr/local/nginx/sbin/nginx  /usr/local/nginx/sbin/nginx_old

cp objs/nginx /usr/local/nginx/sbin/

ls /usr/local/nginx/sbin/

![](D:\github\jichufuwu\image\Untitled\12.gif)

kill -USR2 `cat /usr/local/nginx/logs/nginx.pid`

netstat -anput | grep nginx

nginx -v

![](D:\github\jichufuwu\image\Untitled\13.gif)