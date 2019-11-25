# mysql的主从

## 优点：

1.进行备份，保证数据的安全

2.可以降低主库的压力  主库写入数据，从库读取数据

一主拖八从

## 复制模式：

1.基于sql语句的复制：也就是说把主库上执行的，命令在从库上执行一遍

2.基于行的复制：也就是说将数据库中的数据按照行来进行复制

3.混合模式：默认是基于sql语句的复制，如果sql语句出现问题，就会基于行的复制

## 主从复制的原理：

​    主库当中插入新的数据，写入到二进制日志中（bin-log），然后从库会生成两个线程，其中一个是i/o线程，区主库中的二进制日志中读取内容，同步到自己的中继日志中（relay-log），之后用另一个线程sql线程，将中继日志中的数据，写入到从库当中

i/o线城市用来获取数据传递数据的

sql线程是用来读取中继日志中的内容的

从节点需要指定主节点的ip host  用户名 密码等内容

## 主从：

vim /etc/my.cnf

log-bin=mysql-bin	#开启二进制

server-id       = 1   #节点标识

systemctl restart  mysqld

mysql  -uroot -p

grant  replication slave on *.* to "slave"@"192.168.10.%" identified by "123.com";

flush  privileges;

show master status;

## 从：

vim  /etc/my.cnf

#log-bin=mysql-bin

relay-log=relay-log-bin	#开启

server-id       = 2

mysql -uroot -p

change master to master_host="192.168.10.1",master_user="slave",master_password="123.com",master_log_file="mysql-bin.000005",master_log_pos=338;

start  slave;

show slave status\G

解决错误：

从： stop slave;  reset  slave;

主：从新授权  查看主的状态

从：接收授权  start  slave;



## 主主：

1.备份	保证数据安全

2.两边都可以写或者读

主主：

主一：

vim  /etc/my.cnf

log-bin=mysql-bin		#二进制日志
log-slave-updates=on	#能让二进制日志同步到中继日志
auto_increment_offset=1	#从1开始自增
auto_increment_increment=2	#每次增加2个

systemctl restart mysqld

主二：

vim  /etc/my.cnf  

log-bin=mysql-bin
log-slave-updates=on
auto_increment_offset=2
auto_increment_increment=2

systemctl restart mysqld

主一：

grant replication slave on *.* to "slave"@"192.168.10.%" identified by "123.com";

flush  privileges;       

show  master status;

主二：

​        mysql  -uroot -p

​        change master to master_host="192.168.10.1",master_user="slave",master_password="123.com",master_log_file="mysql-bin.000006",master_log_pos=348;    

start  slave;

show  slave status\G

同上主二对主一授权                          