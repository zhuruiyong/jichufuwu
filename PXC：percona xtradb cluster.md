## PXC：percona xtradb cluster

percona：基于mysql数据库二次开发的数据库产品
xtradb：存储引擎
mysql：myisam-4版本，innodb-5版本
     要搭建PXC架构至少需要三个mysql实例来组成一个集群，三个实例之间不是主从模式，而是各自为主，所以三者是对等关系，不分主从，这就叫多主架构，客户端写入和读取数据时，连接哪个实例都是一样的，读取到的数据是相同的，写入任意一个实例之后，集群自己会将新写入的数据同步到其他实例上，这种架构不共享任何数据，是一种高冗余架构
实现数据库集群数据同步的强一致性
客户端——写入请求——节点A完成写入——将写入操作广播至集群中的B，C节点——B,C节点收到写入请求时会进行核对，执行这些操作是否会产生数据冲突—如果不会产生冲突，就执行写入操作，并把完成的结果返回给节点A—节点A收到所有节点反馈的完成消息后执行提交操作，将结果返回给客户端
 PXC集群特点
 1、多主架构，集群中没有主从之分，每一个节点都可以写入和读取数据
 2、节点数据强一致性(同步)
 3、并行复制
 4、集群中节点故障在恢复或者添加新节点时，节点会自动同步数据，不需要手动同步
 5、支持增量复制也支持全量复制
gelera：PXC依赖的库文件
wsrepc：PXC依赖的库文件

PXC使用的端口：
3306：用来提供数据库服务
4567：集群中节点之间通信的端口
4568：节点间进行增量复制（IST）使用的端口
4444：节点间进行全量复制（SST）使用的端口

三台基础环境、连接网络先下载yum源
 yum install -y  http://www.percona.com/downloads/percona-release/redhat/0.1-3/percona-release-0.1-3.noarch.rpm 联网下载yum源
 [root@localhost ~]# cd /etc/yum.repos.d/
 [root@localhost yum.repos.d]# mv data/C* .
 [root@localhost yum.repos.d]# vim percona-release.repo 修改下载后的yum源
  8 gpgcheck = 0
  9 删除
 14 gpgcheck = 0
 15 删除、以下都删除
  [root@localhost yum.repos.d]#yum -y install Percona-XtraDB-Cluster-57

 [root@localhost ~]# cd /etc/percona-xtradb-cluster.conf.d/

######   [root@localhost percona-xtradb-cluster.conf.d]# vim mysqld.cnf

​    6 [mysqld]
​    7 server-id=1    三台服务器的id不能相同
​    8 datadir=/usr/local/pxc/data
​    9 socket=/var/lib/mysql/mysql.sock
   10 log-error=/usr/local/pxc/log/mysqld.log
   11 pid-file=/usr/local/pxc/data/mysqld.pid
   12 log-bin=mysql-bin
添加：
skip-grant-tables  //root没有密码，添加该选项跳过权限认证，使root可以空密码登录，登录到数据库之后对root用户添加密码之后将该选项删除

  [root@localhost percona-xtradb-cluster.conf.d]# mkdir /usr/local/pxc
  [root@localhost percona-xtradb-cluster.conf.d]# mkdir /usr/local/pxc/data
  [root@localhost percona-xtradb-cluster.conf.d]# mkdir /usr/local/pxc/log
[root@localhost percona-xtradb-cluster.conf.d]# vim wsrep.cnf 
wsrep_provider=/usr/lib64/galera3/libgalera_smm.so   //指定依赖的galera库文件
wsrep_cluster_address=gcomm://192.168.2.20,192.168.2.30,192.168.2.40   //指定集群中的节点，gcomm，节点间通信使用的协议
binlog_format=ROW    //行级复制
default_storage_engine=InnoDB    //指定默认使用的存储引擎
wsrep_cluster_name=pxc-cluster    //指定集群名字
wsrep_node_address=192.168.2.20    //指定当前节点的IP
wsrep_node_name=pxc-node-1    //指定当前节点的名字
pxc_strict_mode=PERMISSIVE    //同步数据时模式为宽容模式
wsrep_sst_auth="sstuser:123.com"   //SST(全量备份)时使用的身份验证的用户
添加：
skip-grant-tables  //root没有密码，添加该选项跳过权限认证，使root可以空密码登录，登录到数据库之后对root用户添加密码之后将该选项删除
  [root@localhost percona-xtradb-cluster.conf.d]# chown -R mysql:mysql /usr/local/pxc  以上在三台服务器上的操作
[root@localhost percona-xtradb-cluster.conf.d]# scp mysqld.cnf wsrep.cnf root@192.168.10.20:/etc/percona-xtradb-cluster.conf.d/
  [root@localhost percona-xtradb-cluster.conf.d]# scp mysqld.cnf wsrep.cnf root@192.168.10.30:/etc/percona-xtradb-cluster.conf.d/
后两台
[root@localhost yum.repos.d]# cd /etc/percona-xtradb-cluster.conf.d/
[root@localhost percona-xtradb-cluster.conf.d]# vim mysqld.cnf 
[root@localhost percona-xtradb-cluster.conf.d]# mkdir /usr/local/pxc
[root@localhost percona-xtradb-cluster.conf.d]# mkdir /usr/local/pxc/data
[root@localhost percona-xtradb-cluster.conf.d]# mkdir /usr/local/pxc/log
[root@localhost percona-xtradb-cluster.conf.d]# chown -R mysql:mysql /usr/local/pxc
[root@localhost percona-xtradb-cluster.conf.d]# vim wsrep.cnf 
##启动服务
节点一：
[root@localhost percona-xtradb-cluster.conf.d]# systemctl start mysql@bootstrap
[root@localhost percona-xtradb-cluster.conf.d]# systemctl restart mysql@bootstrap
mysql> update mysql.user set  authentication_string=PASSWORD("123.com") where User="root";
mysql> grant all on *.* to "sstuser"@"localhost" identified by "123.com";
mysql> flush privileges;
mysql> set password=PASSWORD("123.com");
mysql> alter user root@"localhost" password expire never;
mysql> set password for "root"@"localhost"=password("123.com");

节点二：
[root@localhost percona-xtradb-cluster.conf.d]# systemctl start mysqld
节点三：
[root@localhost percona-xtradb-cluster.conf.d]# systemctl start mysqld
验证：
第一台节点上  
因为当开启数据库之后没有权限所以需要在配置文件中添加一行内容
[root@localhost percona-xtradb-cluster.conf.d]# vim mysqld.cnf
skip-grant-tables     #root没有密码  使用这个选项跳过权限认证
[root@localhost ~]# systemctl start mysql@bootstrap
[root@localhost ~]# mysql -uroot -p
mysql> update mysql.user set authentication_string=password("123.com") where User="root"；   #在用户表中插入用户
mysql> flush privileges;   #必须先刷新  否则没有权限授权
Query OK, 0 rows affected (0.11 sec)
mysql> grant all on *.* to "sstuser"@"localhost" identified by "123.com"; #授权给全量复制的用户
mysql> set password=password("123.com");       #给root设置密码  会有警告需要设置密码的时间
ERROR 1133 (42000): Can't find any matching row in the user table
mysql> alter user "root"@"localhost" password expire never;    #让root密码永久生效
[root@localhost ~]# cd /etc/percona-xtradb-cluster.conf.d/
[root@localhost percona-xtradb-cluster.conf.d]# vim mysqld.cnf 
删除掉跳过权限    15行
[root@localhost ~]# systemctl restart mysql@bootstrap
[root@localhost percona-xtradb-cluster.conf.d]# cd /usr/local/pxc/data/
[root@localhost data]# mysql -uroot -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
第二台第三台
[root@localhost percona-xtradb-cluster.conf.d]# systemctl start mysqld
[root@localhost percona-xtradb-cluster.conf.d]# mysql -uroot -p
Enter password: 
需要密码登录
第一个节点
mysql> alter user "root"@"localhost" identified by "123.com";   #测试用户
mysql> show databases;
mysql> create database haha;
Query OK, 1 row affected (0.01 sec)
第二个第三个
mysql> alter user "root"@"localhost" identified by "123.com";
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| haha               |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.01 sec)