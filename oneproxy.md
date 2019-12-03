# oneproxy

 tar -zxvf oneproxy-rhel5-linux64-v6.0.0-ga.tar.gz -C /usr/local/

 cd /usr/src/oneproxy/

vim  demo.sh 

export ONEPROXY_HOME=/usr/local/oneproxy

vim oneproxy.service 

ONEPROXY_HOME=/usr/local/oneproxy

chmod a+x demo.sh

chmod a+x oneproxy.service 

cp  oneproxy.service  /etc/init.d/oneproxy

chkconfig  --add oneproxy

systemctl start oneproxy

再打开一个终端

安装mariadb

mysql -uadmin  -pOneProxy  -h192.168.10.4 -P4041

passwd '123.com';

+---------+------------------------------------------+
| TEXT    | PASSWORD                                 |
+---------+------------------------------------------+
| 123.com | 7FB703DA3682A0CCC20168D44E8A7E92FE676A51 |
+---------+------------------------------------------+

cd conf/

vim proxy.conf 

proxy-address            = 192.168.10.4:3307

admin-address            = 192.168.10.4:4041

proxy-master-addresses.1 = 192.168.10.1:3306@server1
proxy-slave-addresses.1 = 192.168.10.2:3306@server1
proxy-slave-addresses.2 = 192.168.10.3:3306@server1

proxy-group-policy=server1:2		读写分离的策略
proxy-group-security=server1:0		安全策略

server1

0策略   什么也不做 使用lua脚本完成各项任务

1策略	read failover对于读的操作，先从master读取，如果master宕机，在slave读取

3. 对于集群  在集群中先择固定的节点作为写入节点，如果不能使用，则是用另一台
4.  对于读  进行随机抽取
5. 随机读写

安全策略：

0	不做任何的配置

1	进行通过proxy使用ddl语句

2	需要使用带有where条件的sql语句

3	设置集群只能只读

proxy-user-list          = test/7FB703DA3682A0CCC20168D44E8A7E92FE676A51

/etc/init.d/oneproxy  stop

/etc/init.d/oneproxy  start

netstat -anpt | grep oneproxy

vim proxy.conf

proxy-charset            = utf-8_chinese_ci

killall -9 oneproxy

/etc/init.d/oneproxy  stop

/etc/init.d/oneproxy  start

去主MySQL

grant all on *.* to "test"@"192.168.10.%" identified by "123.com";

flush privileges;

firefox 127.0.0.1:8080

![](D:\github\jichufuwu\image\Untitled\8.gif)

 验证思路

登陆oneproxy  登陆集群

1.使用test库  创建表  biao1  写入主

2.主宕机  将不能创建表

验证出  主写入数据

3.先从1上面 创建slave1

​	从2上面   创建slave 2

主宕机现在宕机

4.在前面读取  看到两个从库轮询

验证出从读取数据

mysql -uroot -p123.com -h 192.168.10.4 -P3307

show  databases;

use  test

create  table  biao1(id int, name char(4));



vim /usr/local/oneproxy/conf/part.txt 

内容全删

[

{

​    "table"  :"test",

​    "pkey"  :"id",

​    "type"  :"int",

​    "method"  :"hash",

​    "partitions":

​     [

​       {"suffix":"_0","group":"server1"},

​       {"suffix":"_1","group":"server1"},

​       {"suffix":"_2","group":"server2"},

​       {"suffix":"_3","group":"server2"},

​       {"suffix":"_4","group":"server3"},

​       {"suffix":"_5","group":"server3"},

​       {"suffix":"_6","group":"server4"},

​       {"suffix":"_7","group":"server4"}

​      ]

​    },                                                   

 

​    {

​    "table"  :"benet",

​    "pkey"  :"id",

​    "type"  :"int",

​    "method"  :"hash",

​    "partitions":

​     [

​       {"suffix":"_0","group":"server1"},

​       {"suffix":"_1","group":"server1"},

​       {"suffix":"_2","group":"server2"},

​       {"suffix":"_3","group":"server2"},

​       {"suffix":"_4","group":"server3"},

​       {"suffix":"_5","group":"server3"},

​       {"suffix":"_6","group":"server4"},

​       {"suffix":"_7","group":"server4"}

​      ]

​    }

]

mysql -utest -p123.com -h 192.168.10.4 -P3307

use test;

create  table benet(id int);

create table  test(id int);

show tables;

insert into test_0 values(2);

insert into test_0 values(3);

insert into test_1 values(2);

insert into test_1 values(3);

select * from test_0;

select * from test_1;