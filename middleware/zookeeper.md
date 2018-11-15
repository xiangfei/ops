ZooKeeper集群快速搭建
这里有三台服务器：准备搭建kafka和zookeeper集群 

本文使用了3台机器部署ZooKeeper集群，IP和主机名对应关系如下：

169.44.62.139 kafka1.jollychic.com 
169.44.59.138 kafka2.jollychic.com
169.44.62.137 kafka3.jollychic.com



Step1: 配置/etc/hosts （3台一致）
127.0.0.1 localhost.localdomain localhost
169.44.62.139 kafka1.jollychic.com kafka1
169.44.59.138 kafka2.jollychic.com
169.44.62.137 kafka3.jollychic.com

Step2: 配置zookeeper环境变量 （3台一致）
vim /etc/profile.d/zookeeper.sh 
export ZOOKEEPER_HOME=/data/tools/zookeeper-3.4.5
export PATH=$PATH:$ZOOKEEPER_HOME/bin:$ZOOKEEPER_HOME/conf



Step3: 下载解压安装配置




tar -xvf zookeeper-3.4.5.tar -C /data/tools/
cd /data/tools/zookeeper-3.4.5/conf

将zoo_sample.cfg拷贝一份命名为zoo.cfg,这里我拷贝一份命名为：zoo.cfg

cp -r zoo_sample.cfg zoo.cfg
mkdir /data/tools/zookeeper-3.4.5/tmp/logs

vim zoo.cfg
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just
# example sakes.
dataDir=/data/tools/zookeeper-3.4.5/tmp
dataLogDir=/data/tools/zookeeper-3.4.5/tmp/logs
# the port at which the clients will connect
clientPort=2281
#
# Be sure to read the maintenance section of the
# administrator guide before turning on autopurge.
#
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
#autopurge.purgeInterval=1
server.0=kafka1.jollychic.com:2887:3887
server.1=kafka2.jollychic.com:2887:3887
server.2=kafka3.jollychic.com:2887:3887


编辑maid:
vim /data/tools/zookeeper-3.4.5/tmp/myid 
0

Step4:
远程复制分发安装文件
最好是把文件打包scp过去
接下来将上面的安装文件拷贝到集群中的其他机器上对应的目录

scp -P58958 -r zookeeper-3.4.5  jollybi@169.44.62.137:/data/tools/. 
scp -P58958 -r zookeeper-3.4.5  jollybi@169.44.59.138:/data/tools/. 
拷贝完成后修改对应的机器上的myid。例如修改其他两台机器中的myid如下：

echo 1 > /data/tools/zookeeper-3.4.5/tmp/myid 
echo 2 > /data/tools/zookeeper-3.4.5/tmp/myid 

Step5:启动 ZooKeeper集群


在ZooKeeper集群的每个结点上，执行启动ZooKeeper服务的脚本，如下所示：

[jollybi@kafka1 ~]$   /data/tools/zookeeper-3.4.5/bin/zkServer.sh start 
[jollybi@kafka2 ~]$   /data/tools/zookeeper-3.4.5/bin/zkServer.sh start 
[jollybi@kafka3 ~]$   /data/tools/zookeeper-3.4.5/bin/zkServer.sh start 


[jollybi@kafka2 ~]$ jps
123189 QuorumPeerMain
123309 Kafka
1151 Jps