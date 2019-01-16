64位安装参考

http://wenku.baidu.com/view/a49d3ad96f1aff00bed51e94.html


rhel6.1安装Oracle 11g

 

1       安装依赖软件包
# cd /media/RHEL6.1/Packet

# rpm -ivh nss-softokn-freebl-3.12.9-3.el6.i686.rpm glibc-2.12-1.25.el6.i686.rpm

# rpm -ivh libgcc-4.4.5-6.el6.i686.rpm

# rpm -ivh libgcc-4.4.5-6.el6.x86_64.rpm

# rpm -ivh gcc-4.4.5-6.el6.x86_64.rpm

# rpm -ivh gcc-c++-4.4.5-6.el6.x86_64.rpm

# rpm -ivh glibc-2.12-1.25.el6.i686.rpm

# rpm -ivh glibc-2.12-1.25.el6.x86_64.rpm

# rpm -ivh glibc-common-2.12-1.25.el6.x86_64.rpm

# rpm -ivh glibc-devel-2.12-1.25.el6.i686.rpm

# rpm -ivh glibc-devel-2.12-1.25.el6.x86_64.rpm

# rpm -ivh glibc-headers-2.12-1.25.el6.x86_64.rpm

# rpm -ivh elfutils-libelf-0.152-1.el6.i686.rpm

# rpm -ivh elfutils-libelf-0.152-1.el6.x86_64.rpm

# rpm -ivh elfutils-libelf-devel-0.152-1.el6.i686.rpm

# rpm -ivh elfutils-libelf-devel-0.152-1.el6.x86_64.rpm

# rpm -ivh libstdc++-4.4.5-6.el6.i686.rpm

# rpm -ivh libstdc++-4.4.5-6.el6.x86_64.rpm

# rpm -ivh libstdc++-devel-4.4.5-6.el6.i686.rpm

# rpm -ivh libstdc++-devel-4.4.5-6.el6.x86_64.rpm

# rpm -ivh ksh-20100621-6.el6.x86_64.rpm

# rpm -ivh libaio-0.3.107-10.el6.i686.rpm

# rpm -ivh libaio-0.3.107-10.el6.x86_64.rpm

# rpm -ivh libaio-devel-0.3.107-10.el6.i686.rpm

# rpm -ivh libaio-devel-0.3.107-10.el6.x86_64.rpm

# rpm -ivh libtool-ltdl-2.2.6-15.5.el6.i686.rpm

# rpm -ivh libtool-ltdl-2.2.6-15.5.el6.x86_64.rpm

# rpm -ivh ncurses-libs-5.7-3.20090208.el6.i686.rpm

# rpm -ivh ncurses-libs-5.7-3.20090208.el6.x86_64.rpm

# rpm -ivh readline-6.0-3.el6.i686.rpm

# rpm -ivh readline-6.0-3.el6.x86_64.rpm

# rpm -ivh unixODBC-2.2.14-11.el6.i686.rpm

# rpm -ivh unixODBC-2.2.14-11.el6.x86_64.rpm

# rpm -ivh unixODBC-devel-2.2.14-11.el6.i686.rpm

# rpm -ivh unixODBC-devel-2.2.14-11.el6.x86_64.rpm

2       修改内核参数
# vim /etc/sysctl.conf

kernel.shmmni = 4096

kernel.sem = 250 32000 100 128

fs.file-max = 6815744

net.ipv4.ip_local_port_range = 9000 65500

net.core.rmem_default = 4194304

net.core.wmem_default = 262144

net.core.rmem_max = 4194304

net.core.wmem_max = 1048576

fs.aio-max-nr = 1048576

# sysctl -p

3       新建用户和组
3.1     新建用户和组
# groupadd oinstall

# groupadd dba

# useradd -g oinstall -G dba -m oracle

# passwd oracle

3.2     设置环境变量
# export DISPLAY=10.1.7.6:2.0

# xhost +

# su - oracle

$ vi .bash_profile

ORACLE_BASE=/u01/app/oracle

ORACLE_HOME=$ORACLE_BASE/product/11.2.0/dbhome_1

ORACLE_SID=orcl

PATH=$PATH:$HOME/bin:$ORACLE_HOME/bin

export ORACLE_BASE ORACLE_HOME ORACLE_SID PATH

unset USERNAME

#export DISPLAY=":0.0"

$ source .bash_profile

$ su

3.3     新建安装目录并设置权限
# mkdir -p /u01/app/oracle

# chown oracle:oinstall /u01/app/oracle

# chmod -R 777 /u01/app

4       oracle用户设置 Shell限制
# vim /etc/security/limits.conf

oracle           soft    nproc   2047

oracle           hard    nproc   16384

oracle           soft    nofile  1024

oracle           hard    nofile  65536

 

# vim /etc/pam.d/login

session    required     pam_limits.so

 

# vim /etc/profile

if [ \$USER = "oracle" ]; then

  if [ \$SHELL = "/bin/ksh" ]; then

  ulimit -p 16384

  ulimit -n 65536

  else

  ulimit -u 16384 -n 65536

  fi

  umask 022

fi

 

# vim /etc/hosts

127.0.0.1  localhost.localdomain  localhost

::1    localhost6.localdomain6  localhost6

192.168.1.6    shlab06

5       .准备oracle安装文件
先将oracle的安装文件拷贝到/u01/下

# cd /u01/

# unzip linux.x64_11gR2_database_1of2.zip && unzip linux.x64_11gR2_database_2of2.zip

# su - oracle

$ cd /u01/database/

$ ./runInstaller

6       安装Oracle 11g
 

7       随机启动
7.1     建立启动脚本
[root@oracle ~]#vim /etc/rc.d/init.d/oracle

#!/bin/bash  

# chkconfig: 2345 99 10  

# description: Startup Script for oracle Databases  

# /etc/rc.d/init.d/oracle

export ORACLE_BASE=/u01/app/oracle/

export ORACLE_HOME=/u01/app/oracle/product/11.2.0/dbhome_1

export ORACLE_SID=orcl

export PATH=$PATH:$ORACLE_HOME/bin

case "$1" in

start)

echo "-----startup oracle-----" >> /var/log/racle11log

su oracle -c "$ORACLE_HOME/bin/dbstart"

su oracle -c "$ORACLE_HOME/bin/emctl start dbconsole"

touch /var/lock/subsys/oracle

echo "-----startup oracle successful-----" >> /var/log/oraclelog

echo "OK"  

;;

stop)

echo "-----shutdwn oracle-----" >> /var/log/oraclelog

su oracle -c "$ORACLE_HOME/bin/dbshut"

su oracle -c "$ORACLE_HOME/bin/emctl stop dbconsole"

rm -f /var/lock/subsys/oracle

echo "-----shutdown oracle successful-----" >> /var/log/oraclelog

echo "OK"  

;;

*)

echo "Usage: 'basename $0' start|stop"  

exit 1

esac

exit 0

7.2     给脚本设置权限
[root@oracle ~]#chmod 755 /etc/rc.d/init.d/oracle

7.3     建立服务
[root@oracle ~]#chkconfig --add oracle

[root@oracle ~]#chkconfig oracle on

8       初始化Oracle 11g
8.1     创建表空间和用户
# cd /app_data1

# mkdir ora_data

# chown oracle:oinstall ora_data/

# cd /app_data2

# mkdir ora_data

# chown oracle:oinstall ora_data/

# cd /app_log

# mkdir ora_log

# chown oracle:oinstall ora_log/

 

--创建空间

create tablespace lsmp1 datafile '/app_data_1/ora_data/ora_data1.dbf' SIZE 8000M

AUTOEXTEND ON NEXT 500M MAXSIZE unlimited EXTENT MANAGEMENT LOCAL SEGMENT SPACE MANAGEMENT AUTO;

 

alter tablespace lsmp1 add datafile '/app_data_1/ora_data/ora_data2.dbf' size 8000M

AUTOEXTEND ON NEXT 500M MAXSIZE unlimited;

alter tablespace lsmp1 add datafile '/app_data_1/ora_data/ora_data3.dbf' size 8000M

AUTOEXTEND ON NEXT 500M MAXSIZE unlimited;

alter tablespace lsmp1 add datafile '/app_data_1/ora_data/ora_data4.dbf' size 8000M

AUTOEXTEND ON NEXT 500M MAXSIZE unlimited;

 

create tablespace lsmp2 datafile '/app_data_2/ora_data/ora_data1.dbf' SIZE 8000M

AUTOEXTEND ON NEXT 500M MAXSIZE unlimited EXTENT MANAGEMENT LOCAL SEGMENT SPACE MANAGEMENT AUTO;

 

alter tablespace lsmp2 add datafile '/app_data_2/ora_data/ora_data2.dbf' size 8000M

AUTOEXTEND ON NEXT 500M MAXSIZE unlimited;

alter tablespace lsmp2 add datafile '/app_data_2/ora_data/ora_data3.dbf' size 8000M

AUTOEXTEND ON NEXT 500M MAXSIZE unlimited;

alter tablespace lsmp2 add datafile '/app_data_2/ora_data/ora_data4.dbf' size 8000M

AUTOEXTEND ON NEXT 500M MAXSIZE unlimited;

 

--Oracle创建用户权限；并授权

create user dlsp identified by dlsp default tablespace lsmp1;

grant resource,connect,dba to dlsp;

 

--更改redo日志文件目录

select * from v$Log;

select * from v$logfile;

ALTER DATABASE DROP LOGFILE GROUP 4;

ALTER DATABASE ADD LOGFILE GROUP 4 ('/app_log/ora_log/redo04.log') SIZE 200M;

ALTER SYSTEM SWITCH LOGFILE;

ALTER SYSTEM CHECKPOINT;

ALTER DATABASE DROP LOGFILE GROUP 3;

ALTER DATABASE ADD LOGFILE GROUP 3 ('/app_log/ora_log/redo03.log') SIZE 200M;

ALTER DATABASE DROP LOGFILE GROUP 2;

ALTER DATABASE ADD LOGFILE GROUP 2 ('/app_log/ora_log/redo02.log') SIZE 200M;

ALTER DATABASE DROP LOGFILE GROUP 1;

ALTER DATABASE ADD LOGFILE GROUP 1 ('/app_log/ora_log/redo01.log') SIZE 200M;

 

--更改undo日志文件目录

select * from dba_tablespaces;

-- 创建一个新的小空间的undo tablespace

create undo tablespace undotbs2 datafile '/app_log/ora_log/undotbs01.dbf' size 8000M;

-- 设置新的表空间为系统undo_tablespace

alter system set undo_tablespace=undotbs2;

--该数据文件的自动扩展打开,关掉了就可以了 

alter database datafile '/app_log/ora_log/undotbs01.dbf' autoextend off;

-- Drop 旧的表空间

drop tablespace undotbs1 including contents;

8.2     其它操作（按需要操作）
--更改Oracle字符集

SQL>select userenv('language') from dual;

SQL>select nls_charset_name(to_number('0354','xxxx')) from dual;

SQL>SHUTDOWN IMMEDIATE;

SQL>STARTUP MOUNT;

SQL>ALTER SYSTEM ENABLE RESTRICTED SESSION;

SQL>ALTER SYSTEM SET JOB_QUEUE_PROCESSES=0;

SQL>ALTER SYSTEM SET AQ_TM_PROCESSES=0;

SQL>ALTER DATABASE OPEN;

SQL>ALTER DATABASE CHARACTER SET AL32UTF8;

SQL>ALTER DATABASE CHARACTER SET INTERNAL_USE AL32UTF8;

SQL>ALTER DATABASE NATIONAL CHARACTER SET AL32UTF8;

SQL>SHUTDOWN IMMEDIATE;

SQL>STARTUP

 

--删除表空间

drop tablespace lsmp including contents and datafiles;