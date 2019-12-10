## pxe+kickstart   无人值守安装

pxe  预启动环境   可以让计算机在网络中启动操作系统，主要用于安装客户机的引导系统

#### kickstart    无人值守安装的一种方式，其工作原理就是将运维人员的操作，保存在ks.cfg的文件中，在安装过程中自动执行里面的步骤。

dhcp  分配ip  让两个主机能够ping通

tftp  简单的文件传输协议   负责传输较小的文件  其特点不需要通过认证  就可以直接传输  69端口

ftp（httpd）  将完整的操作系统进行传输  通过共享目录的方式

ftp  /var/ftp

httpd  /var/www



## 1.安装dhcp    给其他主机分配ip

 yum -y install  dhcp

 vim /etc/dhcp/dhcpd.conf 

subnet  192.168.10.0  netmask  255.255.255.0 {
        range  192.168.10.50  192.168.10.100;    #地址池
        next-server  192.168.10.1;    #tftp的服务器ip
        filename "pxelinux.0";			#第一个引导文件的名称
}

systemctl restart dhcpd

## 2.安装tftp服务  tftp-server服务包 xineted  tftp管理工具

yum -y install  tftp-server  xinetd

 vim /etc/xinetd.d/tftp 

 14         disable                 = no    开启tftp的服务

## 3.安装syslinux  里面有所需要的引导文件

yum -y install syslinux

cd /var/lib/tftpboot/         #tftp的传输文件目录

[root@localhost tftpboot]# cp   /usr/share/syslinux/pxelinux.0 .		#引导程序

[root@localhost tftpboot]# cp /media/images/pxeboot/vmlinuz   .		#虚拟的根

[root@localhost tftpboot]# cp  /media/images/pxeboot/initrd.img  .		#虚拟内核
[root@localhost tftpboot]# cp  /media/isolinux/vesamenu.c32  .		#菜单
[root@localhost tftpboot]# cp /media/isolinux/boot.*  .			#提示信息
[root@localhost tftpboot]# cp /media/isolinux/splash.png  .				背景图片
[root@localhost tftpboot]# ls
boot.cat  boot.msg  initrd.img  pxelinux.0  splash.png  vesamenu.c32  vmlinuz

[root@localhost tftpboot]# mkdir  pxelinux.cfg  建立配置文件目录

[root@localhost tftpboot]# cp /media/isolinux/isolinux.cfg pxelinux.cfg/default		#将引导程序的配置文件复制并改名  改名为default是为了直接能去安装操作系统

[root@localhost tftpboot]# vim pxelinux.cfg/default 

  1 default linux		#默认去选择直接安装

 64   append initrd=initrd.img inst.stage2=ftp://192.168.10.1/centos ks=ftp://192.168.10.1/centos/ks.cfg quiet   			#指定整个操作系统所需要的文件所在的目录和kickstart的配置文件所在的位置

[root@localhost tftpboot]# systemctl restart xinetd
[root@localhost tftpboot]# systemctl restart tftp

yum -y install  vsftpd

[root@localhost tftpboot]# cd /var/ftp/
[root@localhost ftp]# mkdir centos

[root@localhost ftp]# cd centos/

[root@localhost centos]# cp -r /media/* .   #整个操作系统的服务

[root@localhost centos]# cp /root/anaconda-ks.cfg /var/ftp/centos/ks.cfg			#kickstart的配置文件

[root@localhost centos]# chmod +r  /var/ftp/centos/ks.cfg 

[root@localhost centos]# vim /var/ftp/centos/ks.cfg 

  5 #cdrom
  6 url --url=ftp://192.168.10.1/centos		#使用ftp网络安装

 26 timezone Asia/Shanghai --isUtc		#时间同步

 33 clearpart --all --initlabel 			#清空对磁盘的所有操作

 71 reboot								  # 重启 
 72 eula  --agreed						#同意协议	

systemctl restart vsftpd

测试  网络适配器必须相同    内存2g  不用指定镜像