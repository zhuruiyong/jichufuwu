```
web服务
```

## apache（httpd）

1.很多公司都在使用apache，它比较稳定

2.开源的软件

3.支持很多的模块

4.对于公司而言，可以用来做二开

5.能够更加兼容动态页面

基于http协议

## 安装apache

rpm -qa | grep httpd

rpm -evh httpd

yum -y install pcre pcre-devel openssl-devel apr*

拖软件包，解压

tar -zxvf httpd-2.4.25.tar.gz -C /usr/src/

 cd /usr/src/httpd-2.4.25/

./configure  --prefix=/usr/local/httpd --enable-so --enable-ssl --enable-cgi --enable-rewrite --with-pcre --enable-modules=most --enable-mpms-shared=all --with-mpm-prefork

--prefix=/usr/local/httpd 指定安装的路径

--enable-so	支持动态模块的加载机制

--enable-ssl	支持ssl

--enable-cgi	激活cgi

--enable-rewrite	支持地址重写

--with-pcre	在url地址重写的时候会用到	正则表达式库

--enable-modules=most 	编译进去大部分模块	all代表所有

--enable-mpms-shared=all	把支持mpm的模块都编译进来

--with-mpm-prefork	把mpm的模块编译进来以后，默认使用prefork

make &&  make install

cd /usr/local/httpd/

ls

cd bin/

ls

cp apachectl /etc/rc.d/init.d/httpd

ls /etc/rc.d/init.d/

vim  /etc/rc.d/init.d/httpd

#chkconfig:2435 96 36

chkconfig --add httpd

systemctl restart httpd

curl 127.0.0.1
<html><body><h1>It works!</h1></body></html>
[1]+  完成                  firefox http://127.0.0.1

## apache 主配置文件

vim /usr/local/httpd/conf/httpd.conf 

ServerRoot "/usr/local/httpd"	根路径	也就是所谓的安装

Listen	80	监听端口

Pidfile	指定pid文件的位置

LoadModule autoindex_module modules/mod_autoindex.so	代表加载动态模块文件 如果要php结合 需要加载php.so的文件

User daemon	用户

Group	daemon	组

ServerAdmin	you@example.com	管理员的邮箱

#ServerName www.example.com:80	域名

	<Directory />	页面访问的属性h
	AllowOverride none	.htaccess完全忽略	all不忽略
	     Require all denied		拒绝所有的访问
	     Options Index FollowSymlinks	缺少指定的默认页面	将会以目录的形式显示
	     	FollowSymLinks	是否将符号连接指向文件打开
	</Directory>
DocumentRoot "/usr/local/httpd/htdocs"	站点根目录

<IfModule dir_module>

​	DirectoryIndex	idnex.html 默认索引页

</IfModule>

ErrorLog "logs/error_log" 定义错误日志

LogLevel	warn	等级	ERROR WARN INFO DEBUG 高----低

CustomLog "logs/access_log" common  定义日志的文件名和格式



## 虚拟主机

基于域名的虚拟主机

vim  /etc/hosts

192.168.10.1 www.zry.com

192.168.10.1 www.dabai.com

vim /usr/local/httpd/conf/httpd.conf

最后一行

<VirtualHost 192.168.10.1>
        DocumentRoot    /usr/local/httpd/htdocs/zry
        ServerName www.zry.com
</VirtualHost>
<VirtualHost 192.168.10.1>
        DocumentRoot    /usr/local/httpd/htdocs/dabai
        ServerName www.dabai.com
</VirtualHost>

  cd  /usr/local/httpd/htdocs

mkdir  zry

mkdir dabai

echo  "zry"  >  zry/index.html

echo  "dabai"  >  dabai/index.html

systemctl restart httpd

curl  www.zry.com

curl  www.dabai.com

## 基于端口

vim  /usr/local/httpd/conf/httpd.cof

listen 80

listen 8080

<VirtualHost 192.168.10.1:80>
        DocumentRoot    /usr/local/httpd/htdocs/zry
        ServerName www.zry.com
</VirtualHost>
<VirtualHost 192.168.10.1:8080>
        DocumentRoot    /usr/local/httpd/htdocs/dabai
        ServerName www.dabai.com
</VirtualHost>

systemctl restart httpd

## 基于ip

ifcofig ens33:0 192.168.10.2

vim  /usr/local/httpd/conf/httpd.cof

listen  80

<VirtualHost 192.168.10.1>
        DocumentRoot    /usr/local/httpd/htdocs/zry
        ServerName www.zry.com
</VirtualHost>
<VirtualHost 192.168.10.2:8080>
        DocumentRoot    /usr/local/httpd/htdocs/dabai
        ServerName www.dabai.com
</VirtualHost>

systemctl restart httpd

## apache的三种模式

 

prefork模式	单线程模式

 

当apache服务启动后，mpm_prefork模块会预先创建多个子进程，每个子进程只有一个线程，当接收到客户端的请求后，mpm_prefork将这些请求交给子进程处理，每个子进程只能用于处理单个请求，如果请求超过预先创建的子进程数，就会创建新的子进程。 但是对于高并发的环境不适用    如果请求量不大，处理的速度会非常的快 worker模式   多线程或进程 worker使用了多进程和多线程的 混合模式，worker模式也同样会预派生成一些子进程，然后每个子进程生成一些线程，包括一个监听的线程，每个请求过来会被分配到一个线程来服务，线程比进程更轻量，因此，内存的占用会减少一些，更适用于高并发。 但是他不能主动断开长连接，只有超时后，才会断开长连接 event（worker+epoll） 最新的apache的工作模式，在worker的基础上，它把服务的进程从连接中分离出来，worker模式不同的是它解决了keepalive长连接占用资源的浪费的问题  event工作模式会专门派一些线程来管理keepalive的连接，可以断开连接，更好的兼容了高并发    但是他不能很好的支持https的访问 主进程  --  子进程  --  线程 prefork    一个子进程  --- 一个线程  （一个人干一件事） worker     一个子进程 ----多个线程  （一个人干多件事） event       一个子进程  ---keepalive的进程

 

cd /usr/local/httpd/bin/

 

./apachectl -V

 

显示的工作模式为

 

Server MPM:     worker

 

cd /usr/local/httpd/conf/extra/

 

vim httpd-mpm.conf

​     StartServers             5	#服务器启动时创建子进程的数量    MinSpareServers          5	#空闲子进程的数量    MaxSpareServers         10	#空闲子进程的最大数量    MaxRequestWorkers      250	#最大工作的子进程数量    MaxConnectionsPerChild   0	#子进程连接线程的数量	这里为0	代表这个子进程不会消失 

vim /usr/local/httpd/conf/httpd.conf

 

两个工作模式禁用一个

 

LoadModule mpm_prefork_module modules/mod_mpm_prefork.so

 

\#LoadModule mpm_worker_module modules/mod_mpm_worker.so

 

Include conf/extra/httpd-mpm.conf

 

systemctl restart httpd

 

cd /usr/local/httpd/bin/

 

./apachectl -V	#查看工作模式

 

Server MPM:     prefork