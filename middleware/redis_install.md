RedisCluster安装手册

Version:1.0

       Redis 3.0版本支持服务端集群功能，本手册介绍在同一物理机上部署3(Master)+3(Slave)的一个集群安装过程，真实生产环境上可以采用每台物理机上部署一个Master进程和一个Slave进程，Slave进程是其他物理机上的Master的备机，避免服务器在断电的情况下，配对的Master和Slave都不可用。

部署图
同一物理机部署6个Redis进程，7001、7002、7003端口为Master进程、7004、7005、7006端口为Slave进程。多机部署时注意配对的Master和Slave不在同一物理机上即可。


安装步骤
1.        下载源代码文件redis-3.0.0.tar.gz

2.        解压tar -xzvfredis-3.0.0..tar.gz

3.        进入目录cd redis-3.0.0

4.        编译make

5.        安装 make install

6.        创建6个目录用来存放每个进程的配置文件和数据mkdir7001至7006

7.        cpredis.conf 7001/至7006/文件夹下

至此7001至7006这6个目录下均有redis.conf配置件。

修改配置文件
6个redis.conf配置文件相同，注意修改下黄色字体部分的内容即可，其他都相同

配置选项

值

描述

daemonize

yes

 

logfile

"/opt/log/redis-7001.log"

 

databases

1

 

dir

/opt/oracle/redis-3.0.0/7001

用来存放数据的地方

cluster-enabled

yes

 

cluster-config-file

/opt/oracle/redis-3.0.0/7001/nodes.conf

nodes.conf由redis自己生成

cluster-node-timeout

15000

 

cluster-migration-barrier

1

 

cluster-require-full-coverage

yes

只要集群中有一对Master-Slave不可用时，整个集群不可用

启动Redis
按照如下命令分别启动6个Redis进程

[root@inf-p01 redis-3.0.0]# redis-server7001/redis.conf

启动完毕后，6个Redis进程尚未构成集群。

安装Ruby
1.        下载源代码文件ruby-2.1.6.tar.gz

2.        解压tar -xzvfruby-2.1.6.tar.gz

3.        进入目录cd ruby-2.1.6

4.         运行./configure –prefix=/usr/opt/oracle/ruby

5.        编译make

6.        安装 make install

7.         配置环境变量

/etc/profile文件中增加export PATH=/usr/opt/oracle/ruby/bin:$PATH

8.         执行source /etc/profile，使环境变量生效

9.         验证ruby是否安装成功，执行ruby –v，出现如下输出，即ruby安装OK。

ruby 2.1.6p336 (2015-04-13 revision 50298) [x86_64-linux]

安装redis gem
1.         下载源文件redis-3.2.1.gem

2.         安装 gem install –l redis-3.2.1.gem

 

如果安装redis-3.2.1.gem出错，请执行如下步骤

安装zlib

1.         tar -xzvf zlib-1.2.8.tar.gz

2.         cd zlib-1.2.8

3.         ./configure --prefix=/opt/zlib

4.         make

5.         make install

安装ruby-zlib

1.         cd ruby-2.1.6/ext/zlib

2.         ruby ./extconf.rb --with-zlib-dir=/opt/zlib

3.         make

4.         make install

最后安装redis-3.2.1.gem

gem install -l redis-3.2.1.gem

启动集群
1.         进入src目录 cd /opt/oracle/redis-3.0.0/src

2.         执行./redis-trib.rb create --replicas 1 192.168.126.131:7001 192.168.126.131:7002 192.168.126.131:7003 192.168.126.131:7004 192.168.126.131:7005 192.168.126.131:7006

黄色部分为本机IP，出现如下日志即为集群部署OK

[OK] All nodes agree about slots configuration.

>>> Check for open slots...

>>> Check slots coverage...

[OK] All 16384 slots covered.

多机部署时，只要在其中一台机器上执行./redis-trib.rb就可以了

验证
1.         进入redis控制台redis-cli –c –p 7001

2.         set foo car

3.         get foo

如果能取出数据即集群安装OK。