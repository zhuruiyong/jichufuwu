# 			LVS负载均衡

## nat模式  地址转换  负载均衡服务器作为网关存在  请求和响应都需要经过负载均衡服务器

客户端  192.168.10.10  gw192.168.10.1

负载均衡  IP1：  192.168.10.1  ip2：192.168.2.1

web1：  192.168.2.20   gw192.168.2.1

web2：  192.168.2.30  gw192.168.2.1

## web1：

[root@localhost ~]# yum -y install httpd

[root@localhost ~]# echo "11111"  >  /var/www/html/index.html

[root@localhost ~]# systemctl restart httpd

[root@localhost ~]# curl 127.0.0.1

11111

## web2：

[root@localhost ~]# yum -y install httpd

[root@localhost ~]# echo "22222"  >  /var/www/html/index.html

[root@localhost ~]# systemctl restart httpd

[root@localhost ~]# curl 127.0.0.1

22222

## 负载均衡服务器

[root@localhost ~]# vim  /etc/sysctl.conf 

net.ipv4.ip_forward = 1

[root@localhost ~]# modprobe ip_vs  加载ip_vs模块

[root@localhost ~]# cd /media/Packages/

[root@localhost Packages]# rpm -ivh ipvsadm-1.27-7.el7.x86_64.rpm       #安装ipvsadm的管理工具

[root@localhost Packages]# ipvsadm -A  -t 192.168.10.1:80 -s rr 

-A   添加一个新的集群

-t  使用tcp协议   后面加集群被访问的地址（也就是集群的地址）

-s   调度算法

rr   轮询

[root@localhost Packages]# ipvsadm -a -t 192.168.10.1:80  -r  192.168.2.20:80 -m -w 1

[root@localhost Packages]# ipvsadm -a -t 192.168.10.1:80  -r  192.168.2.30:80 -m -w 1

-a	添加真实的节点

-t 	指定集群的地址

-r	后面跟真实服务器的ip

-m	工作模式nat		-g	DR模式	-i	TUN模式

-w	权重

[root@localhost Packages]# ipvsadm -S	#保存

-A -t localhost.localdomain:http -s rr

-a -t localhost.localdomain:http -r 192.168.2.20:http -m -w 1

-a -t localhost.localdomain:http -r 192.168.2.30:http -m -w 1

[root@localhost Packages]# ipvsadm -ln	#查看
IP Virtual Server version 1.2.1 (size=4096)

Prot LocalAddress:Port Scheduler Flags

  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn

TCP  192.168.10.1:80 rr

  -> 192.168.2.20:80              Masq    1      0          0       

  -> 192.168.2.30:80              Masq    1      0          0 

所有都关闭防火墙和沙河

ipvsadm -d -r 192.168.10.1：80 -t 192.168.20.20：80	#删除真实节点

ipvsadm -D　-r   192.168.10.1：80   #删除集群

客户端去访问

curl 192.168.10.1

11111

curl192.168.10.1

22222

## DR模式	直接路由	要求客户端和后端服务器处于同一个物理网络下服务端将响应直接返回给客户端	不需要经过负载均衡服务器

客户端	192.168.2.100

负载均衡	192.168.2.10	vip	192.168.2.200

web1	192.168.2.20	vip	192.168.2.200

web2	192.168.2.30	vip	192.168.2.200

## 负载均衡

[root@localhost Packages]# cd /etc/sysconfig/network-scripts/

[root@localhost network-scripts]# cp ifcfg-ens33 ifcfg-ens33:0

[root@localhost network-scripts]# vim ifcfg-ens33:0

删除uuid 更改ens33为ens33:0  IP为192.168.2.200

## web1

[root@localhost ~]# cd /etc/sysconfig/network-scripts/

[root@localhost network-scripts]# cp ifcfg-lo ifcfg-lo:0

[root@localhost network-scripts]# vim ifcfg-lo:0

DEVICE=lo:0

IPADDR=192.168.2.200

NETMASK=255.255.255.255

NETWORK=127.0.0.0

systemctl restart network

[root@localhost network-scripts]# vim /etc/sysctl.conf 

net.ipv4.conf.lo.arp_ignore = 1

net.ipv4.conf.all.arp_ignore = 1

net.ipv4.conf.default.arp_ignore = 1

net.ipv4.conf.lo.arp_announce = 2

net.ipv4.conf.all.arp_announce = 1

net.ipv4.conf.default.arp_announce = 2

[root@localhost network-scripts]# route add -host 192.168.2.200 dev lo:0

web2  同web1 配置

## 负载均衡

[root@localhost Packages]# ipvsadm -A -t 192.168.2.200:80 -s rr

[root@localhost Packages]# ipvsadm -a -t 192.168.2.200:80 -r 192.168.2.20:80 -g -w 1

[root@localhost Packages]# ipvsadm -a -t 192.168.2.200:80 -r 192.168.2.30:80 -g -w 1

## 客户端验证

[root@localhost ~]# curl 192.168.2.200

22222

[root@localhost ~]# curl 192.168.2.200

11111