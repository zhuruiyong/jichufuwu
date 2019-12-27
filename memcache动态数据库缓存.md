# 	memcache动态数据库缓存

memcache	 开源 	性能比较好		换届后端服务器的压力，提高用户的访问速度，当服务重启或者服务器宕机，缓存的数据将会丢失。

分布式存储	每个节点互不影响

memcache	（c/s）

memcache客户端：

API：负责接收客户端发来的请求，将客户端的请求交给路由算法

路由算法：hash算法

通讯的通道：libecvent来事件的封装  来交给memcache的服务端

memcache服务端：提供缓存的服务	构建hash换（2^32）也就是所谓的缓存数据有2^32个	缓存的数据是有限的，当缓存的数据达到峰值，则会删除掉原来的，如果是集群，数据会绕着hash换找到距离自己最近的缓存服务器节点，并进行缓存

![](D:\github\jichufuwu\image\Untitled\相关包.gif)

```
[root@localhost ~]# cd /usr/local/php/etc/
[root@localhost etc]# vim php-fpm.conf
151 listen = 192.168.10.2:9000
[root@localhost etc]# vim /usr/local/nginx/conf/nginx.conf
 67             fastcgi_pass   192.168.10.2:9000;
[root@localhost etc]# nginx -s reload
[root@localhost etc]# systemctl restart php-fpm
[root@localhost etc]# cd
[root@localhost ~]# tar -zxf libevent-2.0.22-stable.tar.gz -C /usr/src/
[root@localhost ~]# cd /usr/src/libevent-2.0.22-stable/
[root@localhost libevent-2.0.22-stable]# ./configure && make && make install
[root@localhost libevent-2.0.22-stable]# cd
[root@localhost ~]# tar -zxf memcached-1.5.9.tar.gz  -C /usr/src/
[root@localhost ~]# cd /usr/src/memcached-1.5.9/
[root@localhost memcached-1.5.9]# ./configure --prefix=/usr/local/memcached --with-libevent=/usr/local && make && make install
[root@localhost memcached-1.5.9]# ln -s /usr/local/memcached/bin/* /usr/local/bin/
[root@localhost ~]# memcached -d -l 192.168.10.2 -p 11211 -c 10240 -m 512 -P /usr/local/memcached/memcached.pid -u root
-d	后台
-l	指定memcache的监听ip
-p	指定memcache的监听端口
-c	指定么么擦擦和的支持的最大连接量
-m	指定memcache的内存	单位为M
-P	指定memcache的pid路径
-u	指定memcache的用户
[root@localhost ~]# netstat -anput | grep memcached
tcp        0      0 192.168.10.2:11211      0.0.0.0:*               LISTEN      66445/memcached 
[root@localhost ~]# yum -y install telnet
[root@localhost ~]# telnet 192.168.10.2 11211
Trying 192.168.10.2...
Connected to 192.168.10.2.
Escape character is '^]'.
set	添加键值对，如果键值对已经存在，则会将前面的覆盖
add	添加键值对，如果键值对已经存在，则添加失败
replace	对已经存在的键值对进行替换，如果不存在，则修改失败
append	向已经存在的键值对中的值进行向后添加
delete	删除指定的键值对
get		查看指定的键对应的值
flush_all	清除memcache所有的键值对
quit	退出
格式：	set	键名	键的标识  缓存的时间	值的长度
		值
例子：
set name 0 30 4
haha
STORED
get name
VALUE name 0 4
haha
END
[root@localhost ~]# mysql -uroot -p
mysql> create database abc;
mysql> use abc;
mysql> create table test(id int,name varchar(10));
mysql> insert into test values(1,"one"),(2,"two"),(3,"three"),(4,"four"),(5,"five");
mysql> grant all on abc.test to "roo"@"192.168.10.2" identified by "123.com";
mysql> flush privileges;
#安装memcache的模块
[root@localhost ~]# tar -zxf memcache-2.2.7.tgz -C /usr/src/
[root@localhost ~]# cd /usr/src/memcache-2.2.7/
[root@localhost memcache-2.2.7]# ln -s /usr/local/php/bin/* /usr/local/bin/
[root@localhost memcache-2.2.7]# ln -s /usr/local/php/sbin/* /usr/local/sbin/
[root@localhost memcache-2.2.7]# yum -y install autoconf
[root@localhost memcache-2.2.7]# phpize
[root@localhost memcache-2.2.7]# ./configure --enable-memcache --with-php-config=/usr/local/php/bin/php-config && make && make install
[root@localhost memcache-2.2.7]# cd  /usr/local/php/lib/php/extensions/no-debug-non-zts-20090626/
[root@localhost no-debug-non-zts-20090626]# ls
memcache.so
[root@localhost no-debug-non-zts-20090626]# vim /usr/local/php/php.ini 
extension = memcahe.so
[root@localhost no-debug-non-zts-20090626]# systemctl restart php-fpm
[root@localhost no-debug-non-zts-20090626]# cd /usr/local/nginx/html/
[root@localhost html]# vim index.php 
firefox http://192.168.10.2
```

