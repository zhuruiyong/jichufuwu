# 			redis

非关系型数据库  基于键值对来存储数据

redis主要缓存动态的数据

redis：将动态数据缓存在内存中，通过两个AOF和RDB的持久化方式，写入到当前的硬盘中（数据不会丢失）

## redis特点：

```
1.支持多种数据类型
2.支持分布式存储
3.功能比较丰富
```

## redis的常用命令

```
select  切换数据库	redis默认又16个数据库（0-15）
set	创建键值对	如果键值对存储在	则会覆盖
set	key	value
get	查看指定的键值对
get	key
mset	批量创建键值对
mset	key1	value key2	value
append	对应指定的键的值的内容进行追加
append	key	append	value
keys*	查看多所有数据库中的键
dbsize	统计数据库键中的个数
move	对键值对进行迁移
move	key	db
del		删除指定的键值对	可以是多个
del	key1	key2
flushall	清空数据库中的所有键值对
```



## redis缓存   lnmp

```
[root@localhost ~]# vim /usr/local/php/etc/php-fpm.conf
151 listen = 192.168.10.2:9000
[root@localhost ~]# vim /usr/local/nginx/conf/nginx.conf
 67             fastcgi_pass   192.168.10.2:9000;
[root@localhost ~]# nginx -s reload
[root@localhost ~]# systemctl restart php-fpm
[root@localhost ~]# tar -zxf redis-4.0.6.tar.gz  -C /usr/src/
[root@localhost ~]# mv /usr/src/redis-4.0.6/ /usr/local/redis
[root@localhost ~]# cd /usr/local/redis/
[root@localhost redis]# make && make install
[root@localhost redis]# vim redis.conf 
69 bind 192.168.10.2
 136 daemonize yes		#能够后台运行
[root@localhost redis]# redis-server /usr/local/redis/redis.conf 
62097:C 26 Dec 09:18:50.874 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
62097:C 26 Dec 09:18:50.874 # Redis version=4.0.6, bits=64, commit=00000000, modified=0, pid=62097, just started
62097:C 26 Dec 09:18:50.874 # Configuration loaded
[root@localhost redis]# netstat -anput | grep redis
tcp        0      0 192.168.10.2:6379       0.0.0.0:*               LISTEN      62098/redis-server  
[root@localhost redis]# redis-cli -h 192.168.10.2 -p 6379
192.168.10.2:6379> select 0
OK
192.168.10.2:6379> set name one
OK
192.168.10.2:6379> get name
"one"
192.168.10.2:6379> keys *
1) "name"
192.168.10.2:6379> flushall
OK
192.168.10.2:6379> keys *
(empty list or set)
192.168.10.2:6379> exit
[root@localhost ~]# unzip phpredis-master.zip 
[root@localhost ~]# ln -s /usr/local/php/bin/* /usr/local/bin/
[root@localhost ~]# ln -s /usr/local/php/sbin/* /usr/local/sbin/
[root@localhost ~]# yum -y install autoconf
[root@localhost ~]# cd phpredis-master/
[root@localhost phpredis-master]# ls
common.h   debian.control  mkdeb-apache2.sh  redis_session.c
config.m4  igbinary        php_redis.h       redis_session.h
CREDITS    library.c       README.markdown   serialize.list
debian     library.h       redis.c           tests
[root@localhost phpredis-master]# phpize
[root@localhost phpredis-master]# ./configure --with-php-config=/usr/local/php/bin/php-config && make && make install
[root@localhost phpredis-master]# cd /usr/local/php/lib/php/extensions/no-debug-non-zts-20090626/
[root@localhost no-debug-non-zts-20090626]# ls
redis.so
[root@localhost no-debug-non-zts-20090626]# vim /usr/local/php/php.ini 
最后一行添加
extension = redis.so
[root@localhost no-debug-non-zts-20090626]# systemctl restart php-fpm
[root@localhost no-debug-non-zts-20090626]# firefox http://127.0.0.1/index.php

```

![](D:\github\jichufuwu\image\Untitled\redis.gif)

```
[root@localhost no-debug-non-zts-20090626]# mysql -u root -p
mysql> create database abc;
Query OK, 1 row affected (0.00 sec)

mysql> use abc;
Database changed
mysql> create table test(id int,name varchar(10));
Query OK, 0 rows affected (0.37 sec)

mysql> insert into test values(1,"one"),(2,"two"),(3,"three"),(4,"four"),(5,"five");
Query OK, 5 rows affected (0.01 sec)
Records: 5  Duplicates: 0  Warnings: 0

mysql> select * from test;
mysql> grant all on abc.test to "root"@"192.168.10.2" identified by "123.com";
mysql> flush privileges;
[root@localhost www]# vim /usr/local/nginx/html/index.php
<?php
$redis=new Redis;
$redis->connect("192.168.10.100",6379) or die ("could not connect");
$query="select * from abc.test limit 5";
for ($key=1;$key<=5;$key++) {
        if (!$redis->get($key)) {
                $conn=mysql_connect("192.168.10.100","root","123.com");
                $result=mysql_query($query);
                while ($row=mysql_fetch_assoc($result)) {
                        $redis->set($row["id"],$row["name"]);
                }
                break;
        }
        else {
                $name="redis";
                $data[$key]=$redis->get($key);
        }
}
echo $name;
echo "<br>";
for ($key=1;$key<=5;$key++) {
        echo "id is $key";
        echo "<br>";
        echo "name is $data[$key]";
        echo "<br>";
}
?>
[root@localhost html]# firefox http://192.168.10.2
[root@localhost www]# redis-cli -h192.168.10.2 -p6379
Unrecognized option or bad number of args for: '-h192.168.10.2'
[root@localhost www]# redis-cli -h 192.168.10.2 -p 6379
192.168.10.2:6379> keys *
1) "1"
2) "4"
3) "3"
4) "2"
5) "5"
```

## redis集群

redis可以分布式存储

redis 至少要有三个节点	三个节点还需要备份

每个节点都是平等的	每个都有连接

通过hash槽点来分配数据

16384个槽点	（0-16383）	平均分配到每个节点上，只有节点上面有槽点，才会有数据的存在，主节点上有槽点，备份节点上没有槽点，当主节点宕机时，备份节点将会拥有这些槽点，备份节点会变成主节点。

crc16算法	crc16（key） % 16383  =  1541

数据是跟着槽点走的

## redis集群的部署

```
[root@localhost ~]# killall redis-server
[root@localhost ~]# netstat -anput | grep redis-server
[root@localhost ~]# cd /usr/local/
[root@localhost local]# mkdir cluster
[root@localhost local]# cd cluster/
[root@localhost cluster]# mkdir {7000..7005}
[root@localhost cluster]# cp /usr/local/redis/redis.conf 7000/
[root@localhost cluster]# vim 7000/redis.conf
 92 port 7000	#端口
 672 appendonly yes		#持久话
 676 appendfilename "appendonly-7000.aof"	#持久化文件
 814  cluster-enabled yes
 822  cluster-config-file nodes-7000.conf
 828  cluster-node-timeout 15000	#节点的超时时间
[root@localhost cluster]# cd 7000/
[root@localhost 7000]# cp redis.conf ../7001
[root@localhost 7000]# cp redis.conf ../7002
[root@localhost 7000]# cp redis.conf ../7003
[root@localhost 7000]# cp redis.conf ../7004
[root@localhost 7000]# cp redis.conf ../7005
[root@localhost 7000]# sed -i "s/7000/7001/g" ../7001/redis.conf 
[root@localhost 7000]# sed -i "s/7000/7002/g" ../7002/redis.conf 
[root@localhost 7000]# sed -i "s/7000/7003/g" ../7003/redis.conf 
[root@localhost 7000]# sed -i "s/7000/7004/g" ../7004/redis.conf 
[root@localhost 7000]# sed -i "s/7000/7005/g" ../7005/redis.conf 
[root@localhost 7000]# cat ../7002/redis.conf | grep 7002
port 7002
appendfilename "appendonly-7002.aof"
 cluster-config-file nodes-7002.conf
[root@localhost 7000]# cat ../7001/redis.conf | grep 7001
port 7001
appendfilename "appendonly-7001.aof"
 cluster-config-file nodes-7001.conf
[root@localhost 7000]# cd ..
[root@localhost cluster]# redis-server 7000/redis.conf 
[root@localhost cluster]# netstat -anput | grep redis
[root@localhost cluster]# cd
[root@localhost ~]# ln -s /usr/local/redis/src/redis-trib.rb /usr/bin/
[root@localhost ~]# yum -y install ruby
[root@localhost ~]# gem install redis-3.3.0.gem 
[root@localhost ~]# redis-trib.rb  create --replicas 1 192.168.10.2:7000 192.168.10.2:7001 192.168.10.2:7002 192.168.10.2:7003 192.168.10.2:7004 192.168.10.2:7005
[root@localhost ~]# redis-trib.rb check 192.168.10.2:7000
redis-trib.rb	命令字
create 创建	选项 --replicas	给主节点指定从节点的个数
reshard	槽点从新分配
check	检查集群
info	集群信息
add-node	添加节点	默认为主节点
add-node	--slave	添加从节点
--master-id	指定主节点
del-node	删除节点	如果要删除的是主节点	需要将主节点的槽点释放 才能进行删除
[root@localhost ~]# redis-cli -h 192.168.10.2 -p 7000 -c
192.168.10.2:7000> set name aaaaa
-> Redirected to slot [5798] located at 192.168.10.2:7001
OK
192.168.10.2:7001> set aaaaa qqqqq
-> Redirected to slot [13356] located at 192.168.10.2:7002
OK

```

## 添加从节点

```
[root@localhost ~]# cd /usr/local/cluster/
[root@localhost cluster]# mkdir 7006
[root@localhost cluster]# cp 7000/redis.conf 7006/
[root@localhost cluster]# sed -i "s/7000/7006/g" 7006/redis.conf 
[root@localhost cluster]# redis-server 7006/redis.conf 
[root@localhost cluster]# redis-trib.rb  check 192.168.10.2:7000
复制7000的id  68064748c8d648adb28dda62ac5b383ef73db97d
```



[root@localhost cluster]# redis-trib.rb add-node --slave --master-id 68064748c8d648adb28dda62ac5b383ef73db97d 192.168.10.2:7006 192.168.10.2:7000

添加主节点

[root@localhost cluster]# mkdir 7007

[root@localhost cluster]# cp 7000/redis.conf 7007/

[root@localhost cluster]# sed -i "s/7000/7007/g" 7007/redis.conf 

[root@localhost cluster]# redis-server 7007/redis.conf 

[root@localhost cluster]# redis-trib.rb add-node 192.168.10.2:7007 192.168.10.2:7000

[root@localhost cluster]# redis-trib.rb reshard 192.168.10.2:7000 

How many slots do you want to move (from 1 to 16384)? 4096

What is the receiving node ID? 12095666a553808b2e097c0bae407504a209511e

all

yes

[root@localhost cluster]# redis-trib.rb  check 192.168.10.2:7000

删除从节点

[root@localhost cluster]# redis-trib.rb  check 192.168.10.2:7000

复制7006的id

[root@localhost cluster]# redis-trib.rb del-node 192.168.10.2:7006  fc1efb47570c5f2432f68b789237b7db8113e968

