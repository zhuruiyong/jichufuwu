# keepalived 	保证集群高可用

高并发：	能够同时供多个客户访问

高可用：	防止集群中因为某个节点坏掉，而导致整个集群不能正常的使用	

```
	keepalived起初就是为了搭配lvs，对集群进行健康检查，当后端的集群中有一个服务器宕机，它会将这个服务器在当前的集群中剔除，来保证集群的可用性，当这个服务器恢复之后，又加入到这个集群中。
	实现了vrrp协议
	vrrp	虚拟路由冗余协议
	原理：可以对lvs负载均衡服务器来做节点检查，实现高可用，防止单点故障		在负载均衡的集群中，分为主和备，如果主发生故障，加点将会在集群中选举出来一个主，来接替主的位置，主和从之间会发送特定的消息（这个消息一般为1s），当从接收不到主的消息，就意味着主服务器宕机，然后从将会接替vip来进行工作，从而保证集群的高可用	当主服务器修好以后，就继续主的位置
```

```
客户端	192.168.2.20
负载1  192.168.2.1
负载2  192.168.2.10
web1  192.168.2.30
web2  192.168.2.40
web1:httpd
curl
11111
web2：httpd
curl
22222
```

```
lvs  DR  +  keepalived
负载一：
[root@localhost ~]# modprobe  ip_vs
[root@localhost ~]# yum -y install ipvsadm
[root@localhost ~]# ipvsadm -A -t 192.168.2.100:80 -s rr
[root@localhost ~]# ipvsadm -a -t 192.168.2.100:80 -r 192.168.2.30:80 -g -w 1
[root@localhost ~]# ipvsadm -a -t 192.168.2.100:80 -r 192.168.2.40:80 -g -w 1
[root@localhost ~]# ipvsadm -S
-A -t 192.168.2.100:http -s rr
-a -t 192.168.2.100:http -r 192.168.2.30:http -g -w 1
-a -t 192.168.2.100:http -r 192.168.2.40:http -g -w 1
负载二
[root@localhost ~]# modprobe  ip_vs
[root@localhost ~]# yum -y install ipvsadm
[root@localhost ~]# ipvsadm -A -t 192.168.2.100:80 -s rr
[root@localhost ~]# ipvsadm -a -t 192.168.2.100:80 -r 192.168.2.30:80 -g -w 1
[root@localhost ~]# ipvsadm -a -t 192.168.2.100:80 -r 192.168.2.40:80 -g -w 1
[root@localhost ~]# ipvsadm -S
-A -t 192.168.2.100:http -s rr
-a -t 192.168.2.100:http -r 192.168.2.30:http -g -w 1
-a -t 192.168.2.100:http -r 192.168.2.40:http -g -w 1
[root@localhost ~]# ipvsadm -ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  192.168.2.100:80 rr
  -> 192.168.2.30:80              Route   1      0          0         
  -> 192.168.2.40:80              Route   1      0          0
web1
[root@localhost ~]# cd /etc/sysconfig/network-scripts/
[root@localhost network-scripts]# cp ifcfg-lo ifcfg-lo:0
[root@localhost network-scripts]# vim ifcfg-lo:0
DEVICE=lo:0
IPADDR=192.168.2.100
NETMASK=255.255.255.255
[root@localhost network-scripts]# systemctl restart network
[root@localhost network-scripts]# vim /etc/sysctl.conf 
net.ipv4.conf.all.arp_ignore = 1
net.ipv4.conf.all.arp_announce = 2
net.ipv4.conf.default.arp_ignore = 1
net.ipv4.conf.default.arp_announce = 2
net.ipv4.conf.lo.arp_ignore = 1
net.ipv4.conf.lo.arp_announce = 2
web2
[root@localhost ~]# cd /etc/sysconfig/network-scripts/
[root@localhost network-scripts]# cp ifcfg-lo ifcfg-lo:0
[root@localhost network-scripts]# vim ifcfg-lo:0
DEVICE=lo:0
IPADDR=192.168.2.100
NETMASK=255.255.255.255
[root@localhost network-scripts]# systemctl restart network
[root@localhost network-scripts]# vim /etc/sysctl.conf 
net.ipv4.conf.all.arp_ignore = 1
net.ipv4.conf.all.arp_announce = 2
net.ipv4.conf.default.arp_ignore = 1
net.ipv4.conf.default.arp_announce = 2
net.ipv4.conf.lo.arp_ignore = 1
net.ipv4.conf.lo.arp_announce = 2
```

```
两台负载上负载上
[root@localhost ~]# yum -y install popt-devel kernel-devel openssl-devel
[root@localhost ~]# tar -zxf keepalived-1.2.13.tar.gz -C /usr/src/
[root@localhost ~]# cd  /usr/src/keepalived-1.2.13/
[root@localhost keepalived-1.2.13]# ./configure --prefix=/ --with-kernrl-dir=/usr/src/kernel && make && make install
[root@localhost keepalived-1.2.13]# vim /etc/keepalived/keepalived.conf 
global_defs {		#管理员的邮箱
   notification_email {
     acassen@firewall.loc
     failover@firewall.loc
     sysadmin@firewall.loc
   }
   notification_email_from Alexandre.Cassen@firewall.loc  #邮件的发送者
   smtp_server 192.168.200.1	#邮箱服务器的ip
   smtp_connect_timeout 30		#邮箱服务器超时时间
   router_id LVS_DEVEL		#节点的标识
}
更改
vrrp_instance VI_1 {	#vrrp协议组的名称
    state MASTER		#节点的状态	master主	backup备份
    interface ens33		#用来发送vrrp的网卡
    virtual_router_id 51	#server_id	每个组的id必须一致
    priority 100		#当前节点的优先级  默认100  0-255
    advert_int 1		#vrrp的通告时间 主发送给备的通知时间
    authentication {	#主和备之间的认证
        auth_type PASS		#认证类型	passs
        auth_pass 1111		#使用的密钥
    }
   更改：
 virtual_ipaddress {
        192.168.2.100
    }
}
virtual_server 192.168.2.100 80 {	#虚拟ip的端口
    delay_loop 6	#对后端健康检查时间的间隔
    lb_algo rr		#调度算法	轮询
    lb_kind DR		#DR模式
    nat_mask 255.255.255.0		#子网掩码
    persistence_timeout 0		#会话保持时间
    protocol TCP	#传输协议tcp
 38     real_server 192.168.2.30  80{	#真实节点
 39         weight 1		#节点的权重
 40         connect_port 80		#端口
 41             connect_timeout 3	#等待时间
 42             nb_get_retry 3		#节点连接的次数
 43             delay_before_retry 3   #每隔多久节点建立连接
 44     }
 45     real_server 192.168.2.40  80{
 46         weight 1
 47         connect_port 80
 48             connect_timeout 3
 49             nb_get_retry 3
 50             delay_before_retry 3
 51     }
 把后面的全部删除
 负载一
[root@localhost keepalived-1.2.13]# scp /etc/keepalived/keepalived.conf root@192.168.2.10:/etc/keepalived/keepalived.conf
负载二：
vim /etc/keepalived/keepalived.conf
vrrp_instance VI_1 {
    state BACKUP
    interface ens33
    virtual_router_id 51
    priority 90
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.2.100
    }
负载一二
[root@localhost keepalived-1.2.13]# service keepalived start
Unit keepalived.service could not be found.
Reloading systemd:                                         [  确定  ]
Starting keepalived (via systemctl):                       [  确定  ]
负载一有虚拟ip   负载二  ip  a  没有虚拟IP	把第一台宕掉  会漂移到负载二上
ip a
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:87:29:86 brd ff:ff:ff:ff:ff:ff
    inet 192.168.2.1/24 brd 192.168.2.255 scope global ens33
       valid_lft forever preferred_lft forever
    inet 192.168.2.100/32 scope global ens33
       valid_lft forever preferred_lft forever
       
  客户端验证
[root@localhost ~]# curl 192.168.2.100
22222
[root@localhost ~]# curl 192.168.2.100
11111

```

