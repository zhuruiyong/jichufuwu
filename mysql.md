# mysql

关系型数据库：采用了关系型模型来组织的数据库	其已行和列的形式存储数据，以便于用户理解

1.存储方式：采用表格的形式进行存储

2.存储结构：以结构化进行的构建

3.查询方式：结构化查询语句

4.读写性能：十分强调数据的一致性

## 常见关系型数据库 

Oracle （海量）	DB2  mysql  mariadb  SQL server

## 非关系型数据库  

又称nosql  由于指代非关系型的数据库  分布式存储

1.存储方式：采用键值对的形式进行存储，数据以对象的形式存储

2.存储的结构：结构自由，没有表之间的约束

3.查询方式：通过键值对的形式查询

4.读写性能：无需通过sql的解析，读写的性能很高

Nosql	redis	mongodb 	sqlite

## 安装mysql

tar -zxvf  mysql-5.5.22.tar.gz  -C  /usr/src/

 yum  -y install  gcc* cmake  ncurses-devel bison

cd /usr/src/mysql-5.5.22/

cmake -DCMAKE_INSTALL_PREFIX=/usr/local/mysql -DSYSCONFDIR=/etc  -DDEFAULT_CHARSET=utf8 -DDEFAULT_COLLATION=utf8_general_ci -DWITH_EXTRA_CHARSETS=all

 -DCMAKE_INSTALL_PREFIX=/usr/local/mysql	#安装路径

 -DSYSCONFDIR=/etc 	#配置文件的路径

 -DDEFAULT_CHARSET=utf-8 #指定默认字符集

-DDEFAULT_COLLATION=utf8_general_ci 	#指定默认字符集

-DWITH_EXTRA_CHARSETS=all		#扩展字符集

make &&  make install 

groupadd  mysql

useradd -r -g  mysql  mysql

chown -R mysql:mysql /usr/local/mysql/

cp  /usr/local/mysql/support-files/my-huge.cnf  /etc/my.cnf

/usr/local/mysql/scripts/mysql_install_db  --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data

 cd  /usr/local/mysql/support-files/

cp  mysql.server  /etc/rc.d/init.d/mysqld

chmod  a+x  /etc/rc.d/rc.d/init.d/mysqld

chkconfig --add  mysqld

ln -s usr/local/mysql/bin/* /usr/local/bin/

systemctl start mysqld
service mysqld start

