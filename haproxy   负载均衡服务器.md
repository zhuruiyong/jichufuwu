# haproxy   负载均衡服务器

负载分为：四层和七层

haproxy、nginx：四层和七层都能实现

lvs：只支持四层

## haproxy优点：

```
1.开源免费
2.单进程的工作模式
3.支持拒绝连接   防止ddos的攻击
4.支持透明代理
```

## 四层负载

```
	应用于传输层	基于ip和端口实现	传输速度非常的快  负载均衡服务器不做任何解析，直接将客户端发来的请求发送给后端的服务器， 不安全
```

## 七层负载

```
   应用层	调度器和客户端建立tcp连接，接受请求的内容，通过请求的url来过滤请求的资源，会根据过滤出的请求交给后端比较合适的服务器，可以拒绝连接（空连接），比较安全
```

## 调度算法：

roundrobin		动态轮询		weight	不需要重启haproxy

static-rr		静态轮询		weight	需要重启haproxy

leastconnect	最小链接		

source	源地址散列	源地址hash成一个值   

## 1.haproxy负载httpd

客户端：192.168.2.10

负载：192.168.2.1

web1：192.168.2.20   httpd  11111

web2： 192.168.2.30  httpd  22222

```
负载均衡：
[root@localhost ~]# yum -y install pcre-devel bzip2-devel 
[root@localhost ~]# tar -zxf haproxy-1.4.24.tar.gz  -C /usr/src/
[root@localhost ~]# cd /usr/src/haproxy-1.4.24/
[root@localhost haproxy-1.4.24]# uname -r	#查看内核的版本
3.10.0-862.el7.x86_64
[root@localhost haproxy-1.4.24]# make TARGET=linux310 PREFIX=/usr/local/haproxy	#在编译过程中需要有内核版本	根据自己的来写
[root@localhost haproxy-1.4.24]# make install PREFIX=/usr/local/haproxy
[root@localhost haproxy-1.4.24]# ln -s /usr/lcoal/haproxy/sbin/*  /usr/sbin/
[root@localhost haproxy-1.4.24]# cd examples/
[root@localhost examples]# ls
[root@localhost examples]# mkdir /etc/haproxy
[root@localhost examples]# cp haproxy.cfg  /etc/haproxy/
[root@localhost examples]# cp haproxy.init /etc/init.d/haproxy
[root@localhost examples]# chmod a+x /etc/init.d/haproxy 
[root@localhost examples]# vim /etc/haproxy/haproxy.cfg 
  8         #chroot /usr/share/haproxy	#工作的路径
 21         #redispatch		#服务端保存的客户端cookie，如果服务器的节点坏掉，将会重定向到另一个服务器上
 26 listen webservers 0.0.0.0:80
 27         balance roundrobin
 28         option httpchk  GET  /index.html
 29         server web_one 192.168.2.20:80 check inter 2000  rise 3 fall 3
 33          server web_two 192.168.2.30:80 check inter 2000  rise 3 fall 3
 listen  webservers 0.0.0.0:80  #集群的名称  0.0.0.0 监听所有主机的80端口
 balance roundrobin	#采用动态轮询
 option httpchk GET /index.html  #对后端服务器的就健康检测
 server  添加真实服务器的节点
 web_one  节点名称
 192.168.2.20:80 节点的IP和端口
 check  开启健康检查
 inter 2000  每个2000毫秒去进行一次监测
 rise  3  和后端建立连接成功的次数
 fail  3  和后端建立连接失败的次数
 还可以添加
 server_id  服务器节点的标识	一般放在IP的端口后面
 weight  x  给后端的节点添加权重
 maxconn  xxx  后端服务器的最大连接
 backup  将该节点变成备份节点 当主节点不能使用  用备份进行代替
```

客户端验证

firefox http://192.168.2.1

curl 192.168.2.1

11111

curl 192.168.2.1

22222

web端查看日志

[root@localhost ~]# tail -f /var/log/httpd/access_log 

## haproxy代理mysql

```
后端1
[root@localhost ~]# yum -y install mariadb*
[root@localhost ~]# systemctl restart mariadb
[root@localhost ~]# mysql -uroot -p
MariaDB [(none)]> create database one;
Query OK, 1 row affected (0.00 sec)
MariaDB [(none)]> show databases;
MariaDB [(none)]> grant all  on  *.* to "root"@"192.168.2.%" identified by "123.com";
Query OK, 0 rows affected (0.00 sec)
MariaDB [(none)]> flush privileges;
Query OK, 0 rows affected (0.00 sec)
后端2
[root@localhost ~]# yum -y install mariadb*
[root@localhost ~]# systemctl restart mariadb
[root@localhost ~]# mysql -uroot -p
MariaDB [(none)]> create database two;
Query OK, 1 row affected (0.00 sec)
MariaDB [(none)]> show databases;
MariaDB [(none)]> grant all on *.* to "root"@"192.168.2.%" identified by "123.com";
Query OK, 0 rows affected (0.00 sec)
MariaDB [(none)]> flush privileges;
Query OK, 0 rows affected (0.00 sec)

负载均衡端
[root@localhost examples]# vim /etc/haproxy/haproxy.cfg 
 17         mode    tcp
 18         option  tcplog
 26 listen dbservers 0.0.0.0:3306
 27         balance roundrobin
 28         #option httpchk GET /index.html
 29         server db_one 192.168.2.20:3306 check port 3306 maxconn 100
 30         server db_two 192.168.2.30:3306 check port 3306 maxconn 100
[root@localhost examples]# /etc/init.d/haproxy restart
Restarting haproxy (via systemctl):                        [  确定  ]

客户端
安装mariadb
[root@localhost ~]# yum -y install mariadb*
[root@localhost ~]# mysql -uroot -p123.com -P3306 -h192.168.2.1
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| one                |
| performance_schema |
| test               |
+--------------------+
exit
[root@localhost ~]# mysql -uroot -p123.com -P3306 -h192.168.2.1
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| test               |
| two                |
+--------------------+

```

