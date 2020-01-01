## kafka

组件：
kafka  server ：消息系统中间件，接收生产者产生的消息，接收消费者的订阅，将消息交付给消费者处理
topic = vhost   ：主题  对消息的类型进行分类，一个类型对应一个主体，可以对一个主体划分分区  
partition ：存在消息的载体   类似于rabbitmq当中的队列queue
producer：生产者，将产生消息交付给zookeeper
consumer  group：  这个是逻辑的消费者，每个组中有多个消费的实例，用来处理分区的消息，一个分区中的消息只能有每个消费者组中的一个实例处理，来避免一个消息在消费者组中被重复执行
consumer：消费者实例，对kafka中的消息进行订阅
offset：偏移量    对分区的数据进行标识
如果消息被消费    信息仍然会在append.log中保存两天
zookeeper：键值对的    用来存放meta信息（原始数据，最底层的数据）还有watch发现机制
1.broken  node  registry ：borken注册节点，会生成znode的临时节点保存信息
2.broken  topic   registry：  当一个zookeeper启动时会注册自己持有的topic和partition的信息
3.consumer  and consumer group  ： 主要是用来做负载均衡
4.consumer   id  registry：  每个consumer都有唯一的id号，用来标记消费者的信息
5.consumer  offset  tracking   用来跟踪每个concumer消费的最大的offset
6.partition   owner  registry：  用来标记partition被那个consumer所消费
消息传送的机制：
1.at  most once ：   消息发送最多一次 ，发送一次，无论成败，不在重发
2.at  least  once： 消息至少发送一次 ，如果没有成功则再次发送
3.exactly   once：   消息只发送一次
kafka中的集群
leader     follower
leader做真正工作的，follower复制leader的信息  做备份
有几个队列（partition）就有几个leader
kafka的优点：
1.保证消息先来后到的顺序
2.当消息消费后，不会丢失数据  
3.分布式
4.容量大，相对于其他的消息队列，kafka的容量比较大   kafka的容量取决于硬盘的容量
5.数量的大小  不会影响kafka的性能

## kafka群集

```
[root@localhost ~]# java -version
[root@localhost ~]# tar -zxf zookeeper-3.3.6.tar.gz -C /usr/src/
[root@localhost ~]# cd /usr/src/
[root@localhost src]# mv zookeeper-3.3.6/ /usr/local/zookeeper
[root@localhost src]# cd /usr/local/zookeeper/
[root@localhost zookeeper]# cd conf/
[root@localhost conf]# cp zoo_sample.cfg zoo.cfg	#必须在这个
[root@localhost conf]# vim zoo.cfg 
 2 tickTime=2000		#zookeeper集群中各个节点发送心跳包的时间单位毫秒
  5 initLimit=10		#zookeeper新加入follower初始化的时间  单位个
  8 syncLimit=5			#节点超时等待的时间	单位个

 10 dataDir=/usr/local/zookeeper/data	#保存数据的目录
 11 dataLogDir=/usr/local/zookeeper/datalog	#日志产生的目录
 13 clientPort=2181		#zookeeper对外服务的端口

 14 server.1=192.168.10.10:2888:3888		#节点：节点通讯
 15 server.2=192.168.10.20:2888:3888	
 16 server.3=192.168.10.30:2888:3888
[root@localhost conf]# cd /usr/local/zookeeper/
[root@localhost zookeeper]# mkdir data
[root@localhost zookeeper]# mkdir datalog
[root@localhost zookeeper]# cd data
[root@localhost data]# echo "1" > myid
[root@localhost data]# cat myid 
1
[root@localhost data]# scp -r /usr/local/zookeeper/ root@192.168.10.20:/usr/local/
[root@localhost data]# scp -r /usr/local/zookeeper/ root@192.168.10.30:/usr/local/
覆盖后两台的id
去后两台上
[root@localhost ~]# cd /usr/local/zookeeper/data
[root@localhost data]# echo "2" > myid
[root@localhost data]# cat myid
2
[root@localhost ~]# cd /usr/local/zookeeper/data
[root@localhost data]# echo "3" > myid
[root@localhost data]# cat myid 
3
第一台上：
[root@localhost conf]# cd /usr/local/zookeeper/bin/
[root@localhost bin]# ls
README.txt    zkCli.cmd  zkEnv.cmd  zkServer.cmd  zoo.cfg
zkCleanup.sh  zkCli.sh   zkEnv.sh   zkServer.sh
[root@localhost bin]# ./zkServer.sh restart
[root@localhost bin]# netstat -anput | grep java
[root@localhost bin]# ./zkServer.sh status
后两台：
[root@localhost data]# cd /usr/local/zookeeper/bin/
[root@localhost bin]# ./zkServer.sh start 
[root@localhost bin]# netstat -anput | grep java
[root@localhost bin]# ./zkServer.sh status

有一台是
Mode: leader
两台是
Mode: follower

```



```
[root@localhost ~]# tar -zxf kafka_2.11-1.0.1.tgz -C /usr/src/
[root@localhost ~]# cd /usr/src/
[root@localhost src]# mv kafka_2.11-1.0.1/ /usr/local/kafka
[root@localhost src]# cd /usr/local/kafka/config/
[root@localhost config]# vim server.properties 

 21 broker.id=1		#kafka节点的标识
 31 listeners=PLAINTEXT://192.168.10.10:9092	kafka的端口
 60 log.dirs=/usr/lcoal/kafka/data		#消息记录
 103 log.retention.hours=168		#分区文件中消息保存的时间
 104 message.max.byte=1021000		#生产者推送消息的最大容量	单位字节
 105 default.replication.factor=2		#备份节点的个数
 106 replica.fetch.max.bytes=1024000	#消费者单个消费的最大容量	单位字节
125 zookeeper.connect=192.168.10.10:2181,192.168.10.20:2181,192.168.10.30:2181
#指定zookeeper的节点和ip端口
126 num.rtition=1  #指定每个topic创建一个分区（一个队列）
[root@localhost config]# cd /usr/local/kafka/
[root@localhost kafka]# mkdir data
[root@localhost kafka]# scp -r /usr/local/kafka/ root@192.168.10.20:/usr/local/
[root@localhost kafka]# scp -r /usr/local/kafka/ root@192.168.10.30:/usr/local/
后两台上更改：
[root@localhost bin]# vim /usr/local/kafka/config/server.properties 
21 broker.id=2
31 listeners=PLAINTEXT://192.168.10.20:9092
[root@localhost bin]# vim /usr/local/kafka/config/server.properties 
21 broker.id=3
31 listeners=PLAINTEXT://192.168.10.30:9092
三台都启动
[root@localhost kafka]# cd /usr/local/kafka/bin/
[root@localhost bin]# ./kafka-server-start.sh -daemon ../config/server.properties 
[root@localhost bin]# netstat -anput | grep 9092
验证：
创建topic
[root@localhost bin]# ./kafka-topics.sh --create --zookeeper 192.168.10.10:2181 --partitions 1 --replication-factor=2 --topic logs
[2019-12-30 11:02:02,256] WARN Connected to an old server; r-o mode will be unavailable (org.apache.zookeeper.ClientCnxnSocket)
Created topic "logs"
--create	创建
--zookeeper	指定zookeeper
--partition	创建分区的个数
--replication-factor=2	指定分区备份的个数
[root@localhost bin]# ./kafka-topics.sh  --list --zookeeper 192.168.10.10:2181
查看：
[2019-12-30 11:03:58,378] WARN Connected to an old server; r-o mode will be unavailable (org.apache.zookeeper.ClientCnxnSocket)
logs
第二台模拟生产者
[root@localhost bin]# ./kafka-console-producer.sh -broker-list 192.168.10.10:9092 --topic logs
第一台：
模拟消费者：
[root@localhost bin]# ./kafka-console-consumer.sh --zookeeper 192.168.2.10:2181 --topic logs --from-beginning

```

![](D:\github\jichufuwu\image\Untitled\kafka.gif)