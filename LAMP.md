# LAMP

## L（linux） A（apache） M（MySQL  mariadb）  P（php  python  perl）



## php

是一个多用途脚本语言

可以用来编写程序

主要用来编写web界面的

php  中间件？

​	中间件是一个独立的程序，但是它可以用来连接两个应用程序或系统，从而达到让两个独立的应用程序或者资源进行共享。

## lamp的原理流程：

php是用来连接数据库和apache和mysql

是通过接口连接  cgi（fastcgi）（通用网关接口）

## php安装

yum -y install libxml2-devel gd zlib-devel libjpeg-devel  libpng-devel libXpm-devel xz-devel

libxml2-devel 	php能处理xml的页面

gd 		可以加载图形化界面

zlib-devel 		php支持压缩数据

libjpeg-devel		可以加载jpeg的图片

libpng-devel		可以加载png的图片

libXpm-devel 	可以处理xpm的文件

xz-devel		执行高性能比的压缩

tar -zxvf   php-5.3.28.tar.gz  -C /usr/src/

cd /usr/src/php-5.3.28/

./configure --prefix=/usr/local/php --with-config-file-path=/usr/local/php --with-apxs2=/usr/local/httpd/bin/apxs --with-mysql=/usr/local/mysql --with-mysqli=/usr/local/mysql/bin/mysql_config --with-gd --with-zlib --with-jpeg-dir=/usr/lib --with-png-dir=/usr/lib --enable-mbstring

 --prefix=/usr/local/php   #安装路径
--with-config-file-path=/usr/local/php   #指定php的配置文件路径
--with-apxs2=/usr/local/httpd/bin/apxs   #通过apxs2连接httpd
--with-mysql=/usr/local/mysql   #连接数据库
--with-mysqli=/usr/local/bin/mysql_config   #更稳定连接数据库
--with-gd    #支持图形化界面
--with-zlib    #支持压缩
--with-jpeg-dir=/usr/lib    #支持jpeg的图片加载
--with-png-dir=/usr/lib   #支持png的图片加载
--enable-mbstring    #支持多字节的字符

make

报错的话：

/usr/src/php-5.3.28/Zend/zend_language_parser.h:317 int zendparse(void);改成int zendparse(void*compiler_globals);

不报错继续

make install

 cd /usr/src/php-5.3.28/

 cp  php.ini-development  /usr/local/php/php.ini

 vim /usr/local/php/php.ini 

修改：

short_open_tag = On

default_charset = "utf-8"

vim /usr/local/httpd/conf/httpd.conf

修改：

DirectoryIndex index.php index.html

AddType application/x-httpd-php .php

systemctl restart httpd

访问静态页面

firefox  http://127.0.0.1

访问动态页面

cd  /usr/local/httpd/htdocs/

vim index.php

<?php
phpinfo();
?>

systemctl restart httpd

firefox http://127.0.0.1

访问动态页面 调用数据库（apache和php和mysql整合）

mysql -uroot -p

grant all privileges on *.* to "root"@"localhost" identified by "123.com";

flush privileges;

vim index.php 

<?php
$link=mysql_connect("localhost","root","123.com");
if($link) echo "shujukulianjiechenggong";
mysql_close();
?>

