# 				iSCSi

 iSCSi     网络存储的服务   （ip  scsi）

存储的网络：

DAS：直接连接的存储

```
特点：1.管理的成本比较低
	 2.直接依附在服务器上	存储共享受到限制
	 3.占用CPU
```

NAS：网络连接区域

```
nfs    samba   ftp
通过局域网在多个文件服务器之间达到共享
特点：1.集中管理
	 2.可以跨平台使用
	 3.只适用于局域网内
```

SAN：存储局域网络

```
通过高速的光纤网络连接服务器和存储设备  基于scsi ip实现存储共享
特点：1.服务器和存储设备处于完全独立的状态
	 2.使用光纤来传输数据  可以达到一对多
	 3.成本比较高
```

## iscsi组成：

iscsi  target  描述块设备	服务端（存储服务器）

iscsi  initiator   iscsi的发起端	客户端（服务器）

交换机  路由器  tcp/ip  

iscsi  HBA卡  硬件

## 共享的sdb1分区

服务端：

```
添加一块磁盘  重启
[root@localhost ~]# partprobe /dev/sdb
保存分区
[root@localhost ~]# rpm -qa | grep target
selinux-policy-targeted-3.13.1-192.el7.noarch
targetcli-2.1.fb46-1.el7.noarch
[root@localhost ~]# systemctl start target
[root@localhost ~]# systemctl enable target
Created symlink from /etc/systemd/system/multi-user.target.wants/target.service to /usr/lib/systemd/system/target.service.
[root@localhost ~]# systemctl stop firewalld
[root@localhost ~]# setenforce 0
[root@localhost ~]# targetcli
/> ls
o- / ..................................................................... [...]
  o- backstores .......................................................... [...]  #指定添加的存储的类型
  | o- block .............................................. [Storage Objects: 0]	#块设备	磁盘分区逻辑卷
  | o- fileio ............................................. [Storage Objects: 0]	#相当于虚拟磁盘
  | o- pscsi .............................................. [Storage Objects: 0]	#物理scsi
  | o- ramdisk ............................................ [Storage Objects: 0]	#内存盘
  o- iscsi ........................................................ [Targets: 0]		#命名
  o- loopback ..................................................... [Targets: 0]
/> backstores/block  create disk /dev/sdb1		#添加进分区
Created block storage object disk using /dev/sdb1.
/> iscsi/  create iqn.2019-05.com.server.www:disk	#命名
/> iscsi/iqn.2019-05.com.server.www:disk/tpg1/acls create iqn.2019-05.com.client.www:client1		#写入acl控制列表
/> iscsi/iqn.2019-05.com.server.www:disk/tpg1/luns create /backstores/block/disk	#将磁盘和命名绑定
/> iscsi/iqn.2019-05.com.server.www:disk/tpg1/portals/ delete 0.0.0.0 3260		#删除原来的端口
/> iscsi/iqn.2019-05.com.server.www:disk/tpg1/portals/ create 192.168.10.1 3260		#创建新的端口
/> saveconfig 	保存配置
/> exit  退出 
```

![](D:\github\jichufuwu\image\Untitled\iscsi.gif)

客户端：

```
[root@localhost ~]# rpm -qa |  grep iscsi
[root@localhost ~]# cat /etc/iscsi/initiatorname.iscsi 
InitiatorName=iqn.1994-05.com.redhat:12a666579fae
去服务端：
targrtcli
ls
复制
iqn.2019-05.com.client.www:client1

客户端
[root@localhost ~]# vim /etc/iscsi/initiatorname.iscsi 
修改
InitiatorName=iqn.2019-05.com.client.www:client1

[root@localhost ~]# systemctl start iscsid
[root@localhost ~]# systemctl enable iscsid
[root@localhost ~]# iscsiadm -m discovery -p 192.168.10.1:3260 -t sendtargets
-m 	模式
discovery	查找 
-p   ip和端口 
-t 	  类型
sendtargets  将服务器查找的目标发送给客户端

[root@localhost ~]# iscsiadm -m node -T iqn.2019-05.com.server.www:disk -
l   连接服务端的磁盘

[root@localhost ~]# fdisk -l
[root@localhost ~]# iscsiadm -m node -T iqn.2019-05.com.server.www:disk --op update -n node.startup -v automatic	#开机自动连接
node  节点     -T  指定连接的目标    -l  登入  -u登出  
--op   操作    update   更新   create创建   delete 删除
-n   指定的名字    设置开机自启需要使用    -v   update搭配使用   automatic（自动）
/dev/sdb    /mnt/sdb                            xfs   defaults,_netdev  0  0

```

```
逻辑卷：
[root@localhost ~]# pvcreate /dev/sdb2 /dev/sdb3
  Physical volume "/dev/sdb2" successfully created.
  Physical volume "/dev/sdb3" successfully created.
[root@localhost ~]# vgcreate vg /dev/sdb2 /dev/sdb3
  Volume group "vg" successfully created
[root@localhost ~]# lvcreate -L 1G -n lv vg
  Logical volume "lv" created.
[root@localhost ~]# targetcli 
/> backstores/block/ create lvm  /dev/mapper/vg-lv
/> iscsi/ create iqn.2019-05.com.server.www:lvm
/> iscsi/iqn.2019-05.com.server.www:lvm/tpg1/acls create iqn.2019-05.com.client.www:client1
/> iscsi/iqn.2019-05.com.server.www:lvm/tpg1/luns create /backstores/block/lvm
/> iscsi/iqn.2019-05.com.server.www:lvm/tpg1/portals create 192.168.10.1 3260
客户端：
[root@localhost ~]# iscsiadm -m discovery -p 192.168.10.1:3260 -t sendtargets
[root@localhost ~]# iscsiadm -m node -T iqn.2019-05.com.server.www:lvm -l
[root@localhost ~]# fdisk

```

![](D:\github\jichufuwu\image\Untitled\iscsi lvm.gif)