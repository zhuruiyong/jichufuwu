## MHA		数据库高可用集群

特点：
```
1. 故障转移
2. 保证数据的一致性
   MHA的工作流程：
   当监控到集群中的主库宕机，会尝试获取主库宕机时的二进制文件，获取到之后，找到已经同步最新数据的从库，将把该从库的中继日志传递给其他的从库，来同步数据，保证从库之间的数据一致性，在从库中，选择一台新的主库，并同步之前主库的二进制文件，其余的从库从这个新的主库中同步数据，保证主从的集群能正常的运行
```

实验：

```
4台主机
1个管理节点
3个节点 （一主两从）
管理 192.168.43.112
节点：主：192.168.43.167
      从1：192.168.43.196
     从2：192.168.43.197
一主两从要搭建好，
修改从节点上的内容（2个）
[root@localhost ~]# vim  /etc/my.cnf
relay_log_purge=0
read_only=1    #一主一从不用添加，一主多从需要添加，防止从库变为主库，删除自己的中继日志
[root@localhost ~]# systemctl   restart   mysqld
三台mysql上授权
[root@localhost ~]# mysql  -u root  -p
mysql> grant  all  on  *.*  to  "mha"@"192.168.43.%" identified  by  "123.com";
mysql> grant  replication  slave  on  *.*  to  "slave"@"192.168.43.%" identified  by  "123.com";
mysql> flush  privileges;
[root@localhost ~]# ln  -s  /usr/local/mysql/bin/*  /usr/bin/
四台机子的免密登录
[root@localhost ~]# ssh-keygen
[root@localhost ~]# ssh-copy-id root@192.168.43.167 #写其他三台ip
[root@localhost ~]# ssh-copy-id root@192.168.43.196
[root@localhost ~]# ssh-copy-id root@192.168.43.112
安装node：  四台都要安装
[root@localhost ~]# rpm -ivh epel-release-latest-7.noarch.rpm 
[root@localhost ~]# cd /etc/yum.repos.d/
[root@localhost yum.repos.d]# mv data/C* .
[root@localhost yum.repos.d]# yum -y install perl-DBD-mysql perl-DBI
[root@localhost yum.repos.d]# cd 
[root@localhost ~]# rpm -ivh mha4mysql-node-0.56-0.el6.noarch.rpm 
管理节点上安装  没有mysql的节点上
[root@localhost ~]# yum -y install perl-Config-Tiny perl-Log-Dispatch perl-Parallel-ForkManager
[root@localhost ~]# rpm -ivh mha4mysql-manager-0.56-0.el6.noarch.rpm
[root@localhost ~]# vim /etc/masterha_default.cnf
[server default]
user=mha		#管理节点的用户
password=123.com		#用户的密码
repl_user=slave			#主从复制的用户
repl_password=123.com	#用户密码
ssh_user=root		#ssh用户
master_binlog_dir=/usr/local/mysql/data		#主库二进制文件存放的目录	看自己安装的mysql的数据文件在哪里
remote_workdir=/data/login		#主节点宕机之后，获取到二进制文件保存的目录
ping_interval=2				#每隔多长时间进行检测	单位s
shutdown_script=""			#主机欸但那宕机后执行的脚本	可以为空
[root@localhost ~]# mkdir /etc/mha
[root@localhost ~]# vim /etc/mha/app1.cnf
[server default]
manager_workdir=/var/log/manager	#节点工作的日志
manager_log=/var/log/manager/manager.log		#日志文件
[server1]		#监控节点
hostname=192.168.43.167
port=3306
ssh_root=22
[server2]
hostname=192.168.43.196
port=3306
ssh_port=22
candidate_master=1		#当主节点宕机，由这个从节点作为主
[server3]
hostname=192.168.43.197
ssh_root=22

#测试manager与数据库节点之间的ssh通讯是否正常
[root@localhost ~]# masterha_check_ssh --global-conf=/etc/masterha_default.cnf --conf=/etc/mha/app1.cnf
#检测节点之间的主从复制是否正常
验证过程：
mha服务启动：
[root@localhost ~]# masterha_manager --global-conf=/etc/masterha_default.cnf --conf=/etc/mha/app1.cnf
主：
[root@localhost ~]# systemctl stop mysqld
从：
mysql> create database hehe;
Query OK, 1 row affected (0.00 sec)
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| hehe               |
| mysql              |
| performance_schema |
| test               |
+--------------------+


```

