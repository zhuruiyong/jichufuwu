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