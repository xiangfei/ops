1 os   /etc/sysctl.conf
net.ipv4.tcp_tw_reuse = 1 表示开启重用,这是个BOOLEAN。允许将TIME-WAIT sockets重新用于新的TCP连接，默认为0，表示关闭；
net.ipv4.tcp_tw_recycle = 1 表示开启TCP连接中TIME-WAIT sockets的快速回收,这是个BOOLEAN，默认为0，表示关闭。
net.ipv4.tcp_fin_timeout = 30 表示如果套接字由本端要求关闭，这个参数决定了它保持在FIN-WAIT-2状态的时间。单位为秒。
   /etc/security/limits.conf

* soft nofile 65536
* hard nofile 65536
ulimits


2、loadrunner的优化
   a、loadrunner：需要修改windows的注册表参数
    HKLM\System\CurrentControlSet\Services\Tcpip\Parameters
    包含以下：
    EnableTCPA=0
    EnableTCPChimney=0
    ForwardBroadcasts=0
    IPEnableRouter=0
    TcpTimedWaitDeplay=5
    MaxUserPort=65534
   b、去掉controller中的web diagnostics


3.  java monitor
  jstat --gc  

  jstack   


4  look   tcp 
[root@wxlab55 ~]# netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'
TIME_WAIT 24835
CLOSE_WAIT 7
FIN_WAIT1 6
SYN_SENT 112
ESTABLISHED 763
SYN_RECV 2
LAST_ACK 6