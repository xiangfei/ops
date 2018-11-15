    




  下载LOFTER我的照片书  |
Kafka集群快速搭建
kafka集群和zookeeper集群规范


kafka集群和zookeeper集群规范
kafka版本：kafka_2.10-0.8.2.1
zookeeper版本：zookeeper-3.4.5

启动用户：jollybi
kafka安装目录：/data/tools/
kafka新建消息目录：/data/tools/kafka_2.10-0.8.2.1/kafka-logs

启动用户：jollybi
zookeeper安装目录：/data/tools/
zookeeper新建日志目录：/data/tools/zookeeper-3.4.5/tmp/logs

统一配置hosts
169.44.62.139 kafka1.jollychic.com
169.44.59.138 kafka2.jollychic.com
169.44.62.137 kafka3.jollychic.com



本文使用了3台机器部署Kafka集群，IP和主机名对应关系如下：
169.44.62.139 kafka1.jollychic.com 
169.44.59.138 kafka2.jollychic.com
169.44.62.137 kafka3.jollychic.com

Step1: 配置/etc/hosts （3台一致）
127.0.0.1 localhost.localdomain localhost
169.44.62.139 kafka1.jollychic.com kafka1
169.44.59.138 kafka2.jollychic.com
169.44.62.137 kafka3.jollychic.com


Step2: KafKa集群搭建
下载地址：http://mirrors.hust.edu.cn/apache/kafka/0.9.0.0/kafka_2.10-0.8.2.1.tgz
[jollybi@kafka1 ]# tar -xvf kafka_2.10-0.8.2.1.tgz -C /data/tools/
cd /data/tools/kafka_2.10-0.8.2.1/config

#设置data目录，最好不要用默认的/tmp/kafka-logs
mkdir -p /data/tools/kafka_2.10-0.8.2.1/kafka-logs/ 
修改kafka配置文件

[root@kafka_01 config]# vim server.properties

#设置brokerid（从0开始，3个节点分别设为1,2,3，不能重复）在这里id=0跟zookeeper id设置一样就行。 集群机器：按顺序写1
broker.id=1  

auto.leader.rebalance.enable=true 

#修改本地IP地址：
listeners=PLAINTEXT://10.46.72.172:9092
port=9292 //设置访问端口
host.name=169.44.62.139   kafka本机IP
log.dirs=/data/tools/kafka_2.10-0.8.2.1/kafka-logs

#设置注册地址（重要，默认会把本机的hostanme注册到zk中，客户端连接时需要解析该hostanme，所以这里直接注册本机的IP地址，避免hostname解析失败，报错java.nio.channels.UnresolvedAddressException或java.io.IOException: Can not resolve address）
#设置zookeeper地址
#zookeeper.connect=10.46.72.172:2181,10.47.88.103:2181,10.47.102.137:2181
zookeeper.connect=kafka1.jollychic.com:2281,kafka2.jollychic.com:2281,kafka3.jollychic.com:2281

//这里我设置hosts代替IP
配置zookeeper地址

vim zookeeper.properties 
dataDir=/data/tools/zookeeper-3.4.5/tmp
# the port at which the clients will connect
clientPort=2281
# disable the per-ip limit on the number of connections since this is a non-production config
maxClientCnxns=0
~                 
配置kafka访问地址

vim producer.properties 
metadata.broker.list=169.44.62.139:9292,169.44.59.138:9292,169.44.62.137:9292   
# name of the partitioner class for partitioning events; default partition spreads data randomly
#partitioner.class=

# specifies whether the messages are sent asynchronously (async) or synchronously (sync)
producer.type=sync

# specify the compression codec for all data generated: none, gzip, snappy, lz4.
# the old config values work as well: 0, 1, 2, 3 for none, gzip, snappy, lz4, respectively
compression.codec=none

# message encoder
serializer.class=kafka.serializer.DefaultEncoder


Step3: 集群机器快速拷贝配置:


远程复制分发安装文件
最好是把文件打包scp过去
接下来将上面的安装文件拷贝到集群中的其他机器上对应的目录

scp -P58958 -r kafka_2.10-0.8.2.1 jollybi@169.44.62.137:/home/jollybi/tools/. 
scp -P58958 -r kafka_2.10-0.8.2.1 jollybi@169.44.62.138:/home/jollybi/tools/. 
统一修改分别其他两台kafka server.properties

broker.id=2  
host.name=169.44.62.137   kafka2本机IP

broker.id=3 
host.name=169.44.62.138   kafka3本机IP

Step4:启动 kafka集群
./kafka-server-start.sh -daemon ../config/server.properties

[jollybi@kafka1 config]$ jps
3443 QuorumPeerMain
16280 Jps
3628 Kafka

Step5: Kafka常用命令(普及)
Kafka常用命令 以下是kafka常用命令行总结：



1.查看topic的详细信息  
./kafka-topics.sh -zookeeper 127.0.0.1:2181 -describe -topic testKJ1  
2、为topic增加副本  
./kafka-reassign-partitions.sh -zookeeper 127.0.0.1:2181 -reassignment-json-file json/partitions-to-move.json -execute  
3、创建topic 
./kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic testKJ1  
4、为topic增加partition  
./bin/kafka-topics.sh –zookeeper 127.0.0.1:2181 –alter –partitions 20 –topic testKJ1  
5、kafka生产者客户端命令  
./kafka-console-producer.sh --broker-list localhost:9092 --topic testKJ1  
6、kafka消费者客户端命令  
./kafka-console-consumer.sh -zookeeper localhost:2181 --from-beginning --topic testKJ1  
7、kafka服务启动  
./kafka-server-start.sh -daemon ../config/server.properties   
8、下线broker  
./kafka-run-class.sh kafka.admin.ShutdownBroker --zookeeper 127.0.0.1:2181 --broker #brokerId# --num.retries 3 --retry.interval.ms 60 
shutdown broker  
9、删除topic  
./kafka-run-class.sh kafka.admin.DeleteTopicCommand --topic testKJ1 --zookeeper 127.0.0.1:2181  
./kafka-topics.sh --zookeeper localhost:2181 --delete --topic testKJ1  
10、查看consumer组内消费的offset  
./kafka-run-class.sh kafka.tools.ConsumerOffsetChecker --zookeeper localhost:2181 --group test --topic testKJ1