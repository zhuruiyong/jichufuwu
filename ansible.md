# ansible

## 自动化：

系统自动化	pxe	ks

程序自动化	ansible	saltstack	puppet

代码自动化	Jenkins

监控自动化 	zabbix

## puppet	基于ruby语言开发   linux  Windows      unix	1000台以上

## saltstack	基于python语言开发  比较轻量级	1000台以上	支持统一管理

## ansible	基于python语言开发的	使用ssh协议管理  100台以上

puppet和saltstack    c/s（client/server）需要在客户端和服务端同时安装服务

ansible   无客户端模式 

## 自动化运维工具的作用？

可以实现批量的部署	分发文件	减轻运维人员的工作压力

## ansible的特点

1.无客户端	只需要在主控端安装服务

2.通讯使用的ssh协议

3.主控端分发任务是通过模块来实现的

## ansible的核心组成

1.ansible  core	内核

2.host	inventory	主机清单

3.connection	plugins  		ssh

4.playbook		剧本

5.core  	modules		核心模块

6.custom	modules	自定义模块

## ssh的工作原理 ：

ssh		非对称密钥					公钥和私钥	端口22

私钥加密		公钥解密	

​	在主控端生成一对密钥，把公钥传递到远程的主机上，当主控端想要去连接远程主机时，远程主机hi随机发送一串字符给主控端，主控端使用私钥来把这串字符加密，发送给远程主机，远程主机把加密后的字符用公钥来进行解密，解密出的字符，如果和自己生成字符一致，则主控端验证通过。

主控端   192.168.10.1

远程主机1   192.168.10.2

远程主机2	192.168.10.3

主控端安装ansible

[root@localhost ~]# createrepo   /root/app	#制作yum源

[root@localhost ~]# cd  /etc/yum.repos.d/
[root@localhost yum.repos.d]# vim  ansible.repo

[ansible]
name=ansible
baseurl=file:///root/app
enbaled=1
gpgcheck=0

yum -y install  ansible

[root@localhost yum.repos.d]# ansible --version
ansible 2.4.2.0

yum -y install opensssh openssl-devel

[root@localhost yum.repos.d]# cd  /etc/ansible/	#安装目录

[root@localhost ansible]# ls
ansible.cfg  hosts  roles
[root@localhost ansible]# vim hosts

最后一行

[webserver]
192.168.10.2
192.168.10.3
[dbserver]
192.168.10.3

[root@localhost ansible]# ssh-keygen

[root@localhost ansible]# cd
[root@localhost ~]# ls .ssh/
id_rsa  id_rsa.pub

[root@localhost ~]# ssh-copy-id  root@192.168.10.2

[root@localhost ~]# ssh-copy-id  root@192.168.10.3

免密登陆

[root@localhost ~]# ssh root@192.168.10.2
Last login: Mon Dec  9 11:23:46 2019
[root@localhost ~]# exit
登出
Connection to 192.168.10.2 closed.
[root@localhost ~]# ssh root@192.168.10.3
Last login: Tue Dec 10 20:11:28 2019
[root@localhost ~]# exit
登出
Connection to 192.168.10.3 closed.

验证密钥去客户端

[root@localhost ~]# ls .ssh
authorized_keys
[root@localhost ~]# cat .ssh/authorized_keys 
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDPN8U5jCyYMM7N//KyujlbKAkksVyqlOilPvC5sPzLYQCCg+cky4eYFHYJPFAv+9lKxpVPp+x9C1juK4j20oHIGczufcLzE5BBw5pizDdhtlEo0YOvLNYqr3H4+o1Rrh2YLcrhIP8oR3GnYpXEjNGbwpryyCuemy+Tq8YLQd1EPDVfxXIOxr4lS9vQcLn7wsVnLvh6O8ZTxdz78AeyNC2CpaSF5vfGVJxq/J+oba575LosxNzbEr8C/0vSeL2pfypD8/CePKdcXujQMfZZjjDFluAc0bsEr4Pdf7VR0BIEzOzWUVFQDIw9O+I95aUboLAInRYt+++1UUUEiOuzlMc9 root@localhost.localdomain

模块:

常用模块：

使用的格式：

ansible	hosts	-m	module_name（模块名）-a  job （对主机进行什么操作）

## ansible的执行结果：

绿色：执行成功

红色：执行失败

黄色：执行成功	对远程主机的数据进行修改

紫色：警告

#列出ansibke的所有模块

[root@localhost ~]# ansible-doc  -l

查看模块的帮助信息

[root@localhost ~]# ansible-doc  -s ping

## #模块

## #1.ping	检测和远程主机是否能连通

[root@localhost ~]# ansible  webserver -m ping

[root@localhost ~]# ansible  dbserver -m ping

[root@localhost ~]# ansible all -m ping

## #2.command 模块  在远程主机上执行指定的命令	不能使用特殊的字符   :  |   >  >>

[root@localhost ~]# ansible  all -m command -a "ls /"

参数： chdir  切换目录

[root@localhost ~]# ansible  all -m command -a "chdir=/home    ls"

creates	当指定的文件存在时，命令不执行	当指定文件不存在时，命令执行

removes	当指定的文件存在时，命令执行	当指定文件不存在时，命令不执行

[root@localhost ~]# ansible  all -m command -a "creates=/etc/fstab cat /etc/fstab"

[root@localhost ~]# ansible  all -m command -a "removes=/etc/fstab cat /etc/fstab"

## #3.shell模块	在远程主机上执行命令	可以使用特殊的字符

[root@localhost ~]# ansible all -m shell -a "ls /usr | grep src"

## #4.user模块	管理或者创建远程主机上的用户

参数：

name	指定用户名	如果用户不存在	则创建该用户

[root@localhost ~]# ansible all -m user -a 'name=test'

[root@localhost ~]# ansible all -m shell  -a "tail -1 /etc/passwd"

password	给用户添加密码

[root@localhost ~]# openssl passwd -1 123.com
$1$hjpcFO4k$WUtP4uuPQjH5enfW.PtLS1

[root@localhost ~]# ansible all -m user -a 'name=test password=$1$hjpcFO4k$WUtP4uuPQjH5enfW.PtLS1'

[root@localhost ~]# ansible all -m shell -a "tail -1 /etc/shadow"

uid：	指定用户的uid

group：	指定用户的基本组

groups:	指定用户的附加组

[root@localhost ~]# ansible all -m user -a 'uid=1020 name=test'

[root@localhost ~]# ansible all -m user -a 'group=root name=test'

[root@localhost ~]# ansible all -m user -a 'groups=zry name=test'

state=absent	删除用户

remove=yes	删除用户同时删除家目录

[root@localhost ~]# ansible all -m user -a 'name=test state=absent remove=yes'

## #5.group模块	管理、创建远程主机的组

name	指定组	不存在则创建

gid	修改指定组的gid

state=absent	删除指定的组

[root@localhost ~]# ansible all -m group -a 'name=one'

[root@localhost ~]# ansible all -m group -a 'name=one gid=1030'

[root@localhost ~]# ansible all -m group -a 'name=one state=absent'

## #6.script	在远程主机上执行主控端的脚本

参数	chdir	切换目录

​		creates	文件存在	脚本不执行

​		removes	文件存在	脚本执行

[root@localhost ~]# vim test.sh

#!/bin/bash
cd /usr
ls | grep src
[root@localhost ~]# chmod +x test.sh      

[root@localhost ~]# ansible all -m script -a 'chdir=/root removes=/etc/fstab test.sh'

## #7.copy模块	主控端的文件复制到远程的主机上

src	要复制文件的路径（源文件）

dest	将文件复制到远程主机的路径

[root@localhost ~]# touch  haha

[root@localhost ~]# ansible all -m copy -a "src=/root/haha dest=/"

[root@localhost ~]# ansible all -m copy -a "content='1111\n2222' dest=/haha"

[root@localhost ~]# ansible all -m shell -a "cat /haha"

force=no	当主控端拷贝的文件名和目标名一致，但内容不一致的时候，放弃拷贝

backup=yes	当主控端的拷贝文件文件名和目标名一致时，但是内容不一致，对目标文件进行备份

[root@localhost ~]# ansible all -m copy -a 'src=/root/haha dest=/ force=no'

[root@localhost ~]# ansible all -m copy -a 'src=/root/haha dest=/ backup=yes'

owner：指定文件的属主

group:	指定文件的数组

mode：指定文件的权限

[root@localhost ~]# touch cc

[root@localhost ~]# ansible all -m copy -a 'src=/root/cc  dest=/ owner=zry group=zry mode=755'

[root@localhost ~]# ansible all -m shell -a 'ls -l /cc'

## #8.setup	可以查看远程主机的信息

[root@localhost ~]# ansible all -m setup 

## #9.yum：在远程主机上使用yum安装软件

参数：

state=installed	安装软件包	state=removed	卸载软件包  disable_gpg_check	取消密钥验证sta

[root@localhost ~]# ansible all -m yum -a 'name=samba state=installed'

## #10.service：	管理远程主机上的服务

name	服务名	state：stared开启	restarted重启	reload重新加载 stopped停止	enabled=yes	开机自启

ansible all -m service -a 'name=smb state=started'

[root@localhost ~]# ansible all -m service -a 'name=smb state=started'

## #11.file模块	创建或删除远程主机上的文件/目录

参数 path	指定文件	如果目标主机不存在	则创建

​	state	创建文件的类型	例子	state=touch

​	touch文件	directory目录  link软连接	hard硬链接

​	owner	属主

​	group	属组

​	mode	权限

​	state=absent	删除文件或者目录

[root@localhost ~]# ansible all -m file -a 'path=/heihei state=touch'

[root@localhost ~]# ansible all -m file -a 'path=/hehe state=directory'

[root@localhost ~]# ansible all -m file -a 'path=/hehe owner=zry group=zry mode=755'

硬链接针对文件	软连接针对文件或目录

[root@localhost ~]# ansible all -m file -a 'path=/www state=link src=/hehe'

[root@localhost ~]# ansible all  -m file -a 'path=/eee state=hard src=/heihei'

