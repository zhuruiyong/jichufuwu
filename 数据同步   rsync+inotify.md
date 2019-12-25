# 数据同步 		rsync+inotify

sync：同步

async：异步

rsync：远程同步  可以将数据同步到和他通信的主机上

```
rsync特点：
1.增量复制：第一次会去同步全部的内容	第二次之同步修改过的内容
2.支持匿名复制  也支持身份验证
3.可以镜像目录树，文件系统
rsync用法：
rsync   [选项]	src	root@ip:/dest   push	（传递）
rsync	[选项]	root@ip:/src   /dest  pull	（拉取）
选项：
-a   代表以下所有选项（-v除外）
-r		递归同步
-l	同步链接文件
-o	同步时，不修改文件的属主
-g	同步时，步修改文件的属组
-p	同步时，步修改文件的权限
-z	同步时，对文件进行压缩


-v	同步时，显示同步信息
-delete	同步时，如果目标目录中与源目录中的文件有不一致，自动删除
-L	同步时，如果有链接文件，则同步连接文件的源文件
```

## rsync的实验

```
主机A：192.168.10.1
主机B：192.168.10.10
B主机同步A主机
A主机：
[root@localhost ~]# ssh-keygen		（四次回车）
[root@localhost ~]# ssh-copy-id root@192.168.10.10
[root@localhost ~]# ssh root@192.168.10.10
Last login: Mon Dec  9 12:17:35 2019
[root@localhost ~]# exit
登出
Connection to 192.168.10.10 closed.
[root@localhost ~]# mkdir /test1
[root@localhost ~]# cd /test1/
[root@localhost test1]# rpm -qa | grep rsync
B主机：
[root@localhost ~]# mkdir /test2
[root@localhost ~]# cd /test2/
A主机：
[root@localhost test1]# mkdir  -p  a/b/c
[root@localhost test1]# rsync -rv /test1/* root@192.168.10.10:/test2
B主机：
ls   查看
同步软连接：
A主机：
[root@localhost test1]# touch aa
[root@localhost test1]# ln -s aa ee
[root@localhost test1]# ls
a  aa  ee
[root@localhost test1]# rsync -lv /test1/* root@192.168.10.10:/test2
skipping directory a
aa
ee -> aa

sent 106 bytes  received 38 bytes  288.00 bytes/sec
total size is 2  speedup is 0.01
B主机：
查看   ls
[root@localhost test2]# ls
a  aa  ee
同步权限：
A主机：
[root@localhost test1]# chmod 777  aa
[root@localhost test1]# ls -l aa
-rwxrwxrwx. 1 root root 0 12月 25 09:11 aa
[root@localhost test1]# rsync -pv /test1/*  root@192.168.10.10:/test2
skipping directory a
aa
skipping non-regular file "ee"

sent 96 bytes  received 70 bytes  110.67 bytes/sec
total size is 2  speedup is 0.01
B主机：
查看
[root@localhost test2]# ls -l aa
-rwxrwxrwx. 1 root root 0 12月 25 09:14 aa
同步属主属组:
A主机：
[root@localhost test1]# useradd haha
[root@localhost test1]# chown haha:haha aa
[root@localhost test1]# ls -l aa
-rwxrwxrwx. 1 haha haha 0 12月 25 09:11 aa
[root@localhost test1]# rsync -ogv  /test1/* root@192.168.10.10:/test2
skipping directory a
aa
skipping non-regular file "ee"

sent 118 bytes  received 70 bytes  125.33 bytes/sec
total size is 2  speedup is 0.01
B主机：
[root@localhost test2]# useradd haha
[root@localhost test2]# ls -l aa
-rwxrwxrwx. 1 haha haha 0 12月 25 09:16 aa
--delete
主机B:
[root@localhost test2]# touch cc
[root@localhost test2]# ls
a  aa  cc  ee
主机A：
[root@localhost test1]# rsync -av --delete /test1/ root@192.168.10.10:/test2
sending incremental file list
deleting cc
./

sent 176 bytes  received 24 bytes  133.33 bytes/sec
total size is 2  speedup is 0.01
主机B：
ls
另一种方式去拉取：
主机B：
[root@localhost test2]# ssh-keygen 
[root@localhost test2]# ssh-copy-id root@192.168.10.1
[root@localhost test2]# ssh  root@192.168.10.1
Last login: Wed Dec 25 09:35:41 2019 from 192.168.10.1
[root@localhost ~]# exit
登出
Connection to 192.168.10.1 closed.
主机A：
[root@localhost test1]# touch uu
[root@localhost test1]# ls
a  aa  ee  uu
主机B：
[root@localhost test2]# rsync -av root@192.168.10.1:/test1/  /test2/
receiving incremental file list
./
uu

sent 49 bytes  received 228 bytes  554.00 bytes/sec
total size is 2  speedup is 0.01
[root@localhost test2]# ls
a  aa  ee  uu

```

## inotify

监控的软件

文件系统	 目录  删除  创建  修改（文件的属性）等等操作

```
[root@localhost test1]# cd
[root@localhost ~]# tar -zxf inotify-tools-3.14.tar\(1\)\(1\).gz  -C /usr/src/
[root@localhost ~]# cd /usr/src/inotify-tools-3.14/
[root@localhost inotify-tools-3.14]# ./configure --prefix=/usr/local/inotify && make && make install
[root@localhost inotify-tools-3.14]# ln -s /usr/local/inotify/bin/* /usr/local/bin/
选项：
-m	一直处于监控的状态
-e	递归监控
-q	将监控的信息显示在终端上
-e	指定监控的事件
--format	指定输出的格式
事件：
create	创建
move	移动
delete	删除
modify	修改内容
close_write	修改文件内容
close_nowrite	查看只读文件
输出格式:
%w	产生监控的路径
%f	监控为目录	输出目录名
%e	事件的名称
%T	输出当前的事件
[root@localhost ~]# inotifywait -mrq --format %w%f -e create,delete,close_write /test1/
打开另一个终端
```

![](D:\github\jichufuwu\image\Untitled\inotify.gif)



## 整合	实现实时同步

```
[root@localhost test1]# vim /etc/rsyncd.conf 

port=873		#端口
address=192.168.10.1		#主机ip
uid = root			#运行的用户
gid = root			#运行的组
use chroot = no		#当其他用户访问该程序	没有root的权限
max connections = 0		#没有连接数
pid file = /var/run/rsyncd.pid		#pid文件的路径
exclude = lost+found/		#排除lost+found的内容
transfer logging = yes		#开启日志
log file = /var/lib/rsync.log		#日志存放的目录
timeout = 900			#超时时间
ignore nonreadable = yes		#设定rsync忽略没有权限的用户
dont compress   = *.gz *.tgz *.zip *.z *.Z *.rpm *.deb *.bz2	#同步时这类文件不需要压缩
[test]		#同步模块的名称
        path = /test1		#实际的目录
        comment = test1		#详细说明
        read only = no		#不只是只读
[root@localhost test1]# systemctl start rsyncd
[root@localhost test1]# netstat -anpt | grep 873
tcp        0      0 192.168.10.1:873        0.0.0.0:*               LISTEN      72568/rsync 
[root@localhost /]# vim rsync.sh
#!/bin/bash
path=/test1
client=192.168.10.10
/usr/local/bin/inotifywait -mrq --format %w%f -e create,delete,close_write /test1/ | while read file
do
if [ -f $file ] ;then
        rsync -a --delete $file root@$client:/test2
        else
        cd $path && rsync -a --delete ./ root@$client:/test2
fi
done
[root@localhost /]# chmod +x rsync.sh 
[root@localhost /]# ./rsync.sh &
[2] 73162
oot@localhost /]# touch /test1/4444

主机B：
[root@localhost test2]# ls
4444  a  aa  ee  uu


```

