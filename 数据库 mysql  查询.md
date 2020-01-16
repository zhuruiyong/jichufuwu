# 			数据库	mysql 	查询

## 查询语法

select  * |字段	from  表1[表2...] [where  条件判断] [order by  字段] [group by 字段] [having 条件判断] [limit offset count]

```
where 条件判断，指定查询符合要求得数据
order by 字段	给指定得字段进行排序
group by 字段	给指定得字段进行分组
having  条件判断	继续添加筛选条件
limit offset count	限制输出得条目数量

mysql> create database fruits;
mysql> use fruits;
mysql> create table fruits(
    -> f_id char(10) not null,
    -> s_id int not null,
    -> f_name varchar(100) not null,
    -> f_price decimal(8,2) not null,
    -> primary key(f_id));
Query OK, 0 rows affected (0.00 sec)
mysql> insert into fruits values
    -> ('a1',101,'apple',5.2),
    -> ('b1',101,'blackberry',10.2),
    -> ('bs1',102,'orange',11.2),
    -> ('bs2',105,'melon',8.2),
    -> ('t1',102,'banana',10.3),
    -> ('t2',102,'grape',5.3),
    -> ('o2',103,'coconut',9.2),
    -> ('cO',101,'cherry',3.2),
    -> ('a2',103,'apricot',4.2),
    -> ('l2',104,'lemon',6.4),
    -> ('b2',104,'berry',7.6),
    -> ('m1',106,'mango',15.7),
    -> ('m2',105,'durian',33.7),
    -> ('t4',107,'date',3.6),
    -> ('m3',105,'kiwifruit',11.6),
    -> ('b5',107,'pawpaw',3.6);
Query OK, 16 rows affected (0.01 sec)
Records: 16  Duplicates: 0  Warnings: 0

1.在水果表中找到价格为4.2得水果记录
mysql> select * from fruits where f_price=4.2;
2.在水果表中查找名字为pawpaw得水果记录
mysql> select * from fruits where f_name='pawpaw';
3.在水果表中查看价格在10到20之间得水果，显示水果名字和价格
mysql> select f_name,f_price from fruits where f_price between 10 and 20;
mysql> select f_name,f_price from fruits where f_price>=10 and f_price<=20;
4.在水果表中查看价格高于10且s_id是105得记录
mysql> select * from fruits where f_price>10 and s_id=105;
5.在水果表中查看价格不在5到16之间得水果记录
mysql> select * from fruits where f_price<=5 or f_price>=16;
mysql> select * from fruits where f_price not between 5 and 16;
6.在水果表中查看供应商编号（s_id）是103或者104得记录
mysql> select * from fruits where s_id=103 or s_id=104;
mysql> select * from fruits where s_id in (103,104);
7.在水果表中查看水果得名字以d开头且价格高于10 的水果记录
mysql> select * from fruits where f_price >10 and f_name like 'd%';
mysql> select * from fruits where f_price >10 and f_name regexp '^d';
8.在水果表中查看名字含有r或i或o或p的水果记录
mysql> select * from fruits where f_name regexp '[r,i,o,p]';
9.在水果表中查看名字以d或者b或者c开头的且价格高于8的水果记录
mysql> select * from fruits where f_price>8 and f_name regexp '^[bcd]';
mysql> select * from fruits where f_price>8 and f_name regexp '^b|^c|^d';
```

## order by 字段，给指定的字段进行排序，适合给重复值较少的字段进行排序，可以分为升序（asc）和降序（desc）

给多个字段排序实现数据的最终有序化

## group by 通常和having进行联合使用，表示在分组过后依然有条件进行筛选

分组中通常会出现聚合函数

```
max（）最大值
min（）最小值
count（）求数量
sum（）求和
avg（）求平均值

select 字段 as名字	  给字段起别名，讲过长的字段名，或者操作名进行简化
mysql> select s_id,group_concat(f_name),count(*) from fruits group by s_id;
在水果表中查看提供的水果种类不超过2种的水果信息，显示供应商编号，水果名字
mysql> select s_id,group_concat(f_name) from fruits group by s_id having count(*)<=2;

having和where的区别
1.where顺序高于having
2.where种不能使用聚合函数而having可以

limit  [offset]  count  限制输出的条目数量
offset 表示偏移量，表示从第几条输出
count  表示数量，表示输出多少条

求水果表中价格最高的水果记录
mysql> select * from fruits order by f_price desc limit 1;


```

```
连接查询，查询两个或者两个以上的表中的内容
（1）内连接
内连接使用比较运算符，进行表与表之间的列数据的比较操作，并且显示表中与连接条件相匹配的内容，组成新的内容，相当于只输出符合条件的数据。
表名 [inner] join 表名 on 条件判断
mysql> select suppliers.s_id,s_name,f_name,f_price from suppliers,fruits where fruits.s_id=suppliers.s_id;
mysql> select suppliers.s_id,s_name,f_name,f_price from suppliers inner join fruits on fruits.s_id=suppliers.s_id;
优点：可以只输出符合要求的数据，结果更简洁
缺点：由于只显示符合要求的数据，会造成数据丢失

（2）外连接
	1)左连接
	左连接关键字：left [outer] join 返回包括左表在内的所有记录和右表中连接字段相等的记录
	2）右连接
	右连接关键字：right [outer] join 返回包括右表在内的所有记录和左表中连接字段相等的记录
判断左右表，在连接关键字的左边无论左右链接都为左表，右边为右表

mysql> create table suppliers(
    -> s_id int not null auto_increment,
    -> s_name varchar(50) not null,
    -> s_city char(50) null,
    -> s_zip char(50),
    -> s_call char(50) not null,
    -> primary key(s_id));
Query OK, 0 rows affected (0.01 sec)

mysql> insert into suppliers values( 101,'FastFruits','Tianjin',300000,10100), (103,'ACME','Shanghai',200000,90001),
    -> (104,'FNK Inc','Jinan',543210,81777),
    -> (105,'Good Set','Taiyuan',030000,22222),
    -> (106,'Just Eat Ours','Beijing','010',45678),
    -> (107,'S Inc','Shijiazhuang',450000,33322);
Query OK, 6 rows affected (0.00 sec)


子查询
又叫做嵌套查询，指一个查询语句在另外一个查询语句内部，从4.1之后开始使用

关键字
any		满足其中的一个条件即可输出
mysql> select num from ex1 where num > any (select num2 from ex2);
all		满足所有条件才能输出
mysql> select num from ex1 where num > all (select num2 from ex2);
exists	判断子查询中的内容是否存在，如果存在执行外层查询语句，如果不存在停止外层查询，返回empty set
mysql> select * from fruits where exists (select * from suppliers where s_id=109);
Empty set (0.00 sec)

mysql> select * from fruits where exists (select * from suppliers where s_id=108);
not exists	判断子查询中的内容是否不存在，如果不存在执行外层查询语句，如果存在停止外层查询，返回empty set

找到城市在北京的供应商，查看该供应商提供的水果
mysql> select * from fruits,suppliers where fruits.s_id=suppliers.s_id and s_city='Beijing';
查看每个供应商的价格最高的水果，显示供应商的编号，水果名以及水果价格
mysql> select s_id,group_concat(f_name) as f_name,f_price from fruits where f_price in (select max(f_price) from fruits group by s_id) group by s_id;
mysql> select s_id,group_concat(f_name) as f_name,f_price from fruits where (s_id,f_price) in (select s_id,max(f_price) from fruits group by s_id) group by s_id;
```

```
联合查询
将两个或者两个以上的查询语句的结果合并在一起进行查看
select column * from 表名
union [all]
select column * from 表名
如果合并查询中不加入all将会过滤重复数据，加all显示全部，在联合查询中不需要考虑数据类型因素，只需要将查看的数据进行显示即可，两个语句之间也不包含任何关联关系，在两个或者两个以上的查询语句中，查询的字段数据数量必须一致，否则报错
mysql> select * from fruits
    -> union all
    -> select * from fruits;
mysql> select * from fruits union  select * from fruits;
mysql> select f_name from fruits
    -> union
    -> select s_id from suppliers;
mysql> select s_id,f_name from fruits where f_price=10.2
    -> union
    -> select s_name,s_city from suppliers where s_id=105;
```

