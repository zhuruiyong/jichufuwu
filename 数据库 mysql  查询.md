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

