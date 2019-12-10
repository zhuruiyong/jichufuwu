## 		lepus（天兔）是一套开源的数据库监控平台，目前支持了mysql  Oracle  SQL server  mongodb  redis  等数据库的监控你和警告。

​      只需要经过后端数据库的授权，就可以远程监控。

## 安装lepus并监控mysql

搭建lepus   需要（lamp）

1.安装lamp和依赖关系

yum -y install httpd  php  php-mysql mariadb-server  mariadb-devel  python-devel

2.安装python的基础模块

mysql  mysqldb for  python

[root@localhost ~]# unzip MySQL-python-1.2.5.zip 

[root@localhost ~]# cd MySQL-python-1.2.5/

[root@localhost MySQL-python-1.2.5]# which  mysql_config
/usr/bin/mysql_config

vim site.cfg 

mysql_config = /usr/bin/mysql_config

cd

[root@localhost ~]# tar -zxvf pip-19.3.1.tar.gz 

[root@localhost ~]# cd pip-19.3.1/
[root@localhost pip-19.3.1]# python setup.py build

[root@localhost pip-19.3.1]# python setup.py install

[root@localhost pip-19.3.1]# cd /root/MySQL-python-1.2.5/
[root@localhost MySQL-python-1.2.5]# python setup.py build

[root@localhost MySQL-python-1.2.5]# python setup.py install

cd

[root@localhost ~]# unzip Lepus-3.7.zip 

[root@localhost ~]# cd lepus_v3.7/python/
[root@localhost python]# python test_driver_mysql.py 
MySQL python drivier is ok!

3.安装lepus的采集器

[root@localhost python]# systemctl start mariadb
[root@localhost python]# mysqladmin -uroot password '123.com';

MariaDB [(none)]> create database lepus default character set utf8;
Query OK, 1 row affected (0.01 sec)

MariaDB [(none)]> grant all on lepus.* to 'lepus'@'localhost' identified by '123.com';
Query OK, 0 rows affected (0.00 sec)

[root@localhost python]# cd  ../sql/

[root@localhost sql]# mysql -ulepus -p123.com lepus < lepus_table.sql

[root@localhost sql]# mysql -ulepus -p123.com  lepus  <  lepus_data.sql

[root@localhost sql]# cd  ../python/
[root@localhost python]# chmod  +x install.sh
[root@localhost python]# ./install.sh 

 cd /usr/local/lepus/

[root@localhost lepus]# vim etc/config.ini 

###监控机MySQL数据库连接地址###
[monitor_server]
host="127.0.0.1"
port=3306
user="lepus"
passwd="123.com"
dbname="lepus"

[root@localhost lepus]# lepus start
nohup: 把输出追加到"nohup.out"
lepus server start success!

4.安装web的管理平台

[root@localhost lepus]# cd 
[root@localhost ~]# cd lepus_v3.7/php/
[root@localhost php]# cp -a . /var/www/html/
[root@localhost php]# systemctl starthttpd

[root@localhost php]# cd /var/www/html/

[root@localhost html]# vim application/config/database.php 

$db['default']['hostname'] = '127.0.0.1';
$db['default']['port']     = '3306';
$db['default']['username'] = 'lepus';
$db['default']['password'] = '123.com';
$db['default']['database'] = 'lepus';
$db['default']['dbdriver'] = 'mysql';

[root@localhost html]# systemctl stop firewalld
[root@localhost html]# setenforce 0

firefox http://127.0.0.1

admin

Lepusadmin

