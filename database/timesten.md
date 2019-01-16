TimesTen安装与配置
Contents
1 Linux操作系统安装        2
2. Windows Timesten安装        8




1 Linux操作系统安装
创建用户timesten。
创建/opt/TimesTen目录，owner修改为timesten
如果和jboss安装到一台机器，owner修改为lsmpusr
创建/etc/TimesTen目录，owner修改为timesten
如果和jboss安装到一台机器，owner修改为lsmpusr

解压timesten安装包。用timesten（timesten单独部署）用户或者lsmpusr（timesten和jboss一起部署）登陆：
[timesten@localhost linux8664]$ ./setup.sh

ERROR: The /etc/TimesTen directory needs to be created for the instance registry
       and its ownership and permissions set appropriately.
       Please refer to the installation guide for assistance.
如果有这个问题，创建/etc/TimesTen目录，owner修改为timesten

[timesten@localhost linux8664]$ ./setup.sh 

NOTE: Each TimesTen installation is identified by a unique instance name.
      The instance name must be a non-null alphanumeric string, not longer
      than 255 characters.

Please choose an instance name for this installation? [ tt1121 ] 
Instance name will be 'tt1121'.
Is this correct? [ yes ] 

Of the three components:

  [1] Client/Server and Data Manager
  [2] Data Manager Only
  [3] Client Only

Which would you like to install? [ 1 ] 

Of the following options :

  [1] /home/timesten
  [2] /home/timesten
  [3] Specify a location
  [q] Quit the installation

Where would you like to install the tt1121 instance of TimesTen? [ 1 ] 3
Please specify a directory to install TimesTen? [ /home/timesten ] /opt/TimesTen 
Where would you like to create the daemon home directory? [ /opt/TimesTen/tt1121/info ] 

The daemon logs will be located in /opt/TimesTen/tt1121/info
Would you like to specify a different location for the daemon logs? [ no ] 
Installing into /opt/TimesTen/tt1121 ...
Uncompressing ...

NOTE: If you are configuring TimesTen for use with Oracle Clusterware, the
      daemon port number must be the same across all TimesTen installations
      managed within the same Oracle Clusterware cluster.

NOTE: All installations that replicate to each other must use the same daemon
      port number that is set at installation time. The daemon port number can
      be verified by running 'ttVersion'.

The default port number is 53388.

Do you want to use the default port number for the TimesTen daemon? [ yes ] 
The daemon will run on the default port number (53388).

NOTE: For security, we recommend that you restrict access to the
      TimesTen installation to members of a single OS group. Only members of
      that OS group will be allowed to perform direct mode connections to
      TimesTen, and only members of that OS group will be allowed to perform
      operations that access TimesTen data stores, TimesTen files and shared
      memory. The OS group defaults to the primary group of the instance
      administrator. You can default to this group, choose another OS group
      or you can make this instance world-accessible. If you choose to make
      this instance world-accessible, all database files and shared memory
      are readable and writable by all users.

Restrict access to the the TimesTen installation to the group 'timesten'? [ yes ] 

NOTE: Enabling PL/SQL will increase the size of some TimesTen libraries.

Would you like to enable PL/SQL for this instance? [ yes ] 

In order to use the 'In-Memory Database Cache' feature in any databases
created within this installation, you must set a value for the TNS_ADMIN
environment variable. It can be left blank, and a value can be supplied later
using <install_dir>/bin/ttModInstall.

Please enter a value for TNS_ADMIN (s=skip)? [  ] s


NOTE: It appears that you are running version 4.1 of the g++
      compiler. TimesTen ships with multiple sets of client libraries and server
      binaries : one built for compatibility with g++ 3.4.6 and one with
      g++ 4.1.0. The installer has created links to the 4.1.0 library in the
      <install_dir>/lib directory and to the 4.1.0 server binary in the
      <install_dir>/bin directory. If you want to use a different compiler,
      please modify the links to point to the desired library and server binary.

Installing server components ...
What is the TCP/IP port number that you want the TimesTen Server to listen on? [ 53389 ] 
Do you want to install QuickStart and the TimesTen Documentation? [ no ] yes
Where would you like to install the quickstart and doc directories (s=skip)? [ /opt/TimesTen/tt1121 ] 

The TimesTen Quickstart applications can take up to 64 Mbytes of disk space.
Depending on how your system is configured, you may not want to create the
QuickStart DemoDataStore directory in the default location,
/opt/TimesTen/tt1121/info/DemoDataStore

Where would you like to create the DemoDataStore directory? [ /opt/TimesTen/tt1121/info ] 
Creating /opt/TimesTen/tt1121/info/DemoDataStore ...

Installing client components ...

Would you like to use TimesTen Replication with Oracle Clusterware? [ no ] 

NOTE: The TimesTen daemon startup/shutdown scripts have not been installed.

Run the 'setuproot' script :
        cd /opt/TimesTen/tt1121/bin
        ./setuproot -install
This will move the TimesTen startup script into its appropriate location.

The startup script is currently located here :
  '/opt/TimesTen/tt1121/startup/tt_tt1121'.

The Quickstart home page can be accessed here :
  '/opt/TimesTen/tt1121/quickstart/index.html'

The 11.2.1.7 Release Notes are located here :
  '/opt/TimesTen/tt1121/README.html'

Starting the daemon ...
Daemon port 53388 already in use:
TimesTen Daemon Release 11.2.1.7, instance tt1121, pid 10968

Unable to start the daemon.
Do you want to abort this installation? [ yes ] 

NOTE:  The installation of instance tt1121 will remain.
       To remove this installation, run '/opt/TimesTen/tt1121/bin/setup.sh -uninstall'

安装后操作，root用户下执行：
./setuproot -install
Would you like to install the TimesTen daemon startup scripts into /etc/init.d? [ yes ] yes
Copying /opt/TimesTen/tt1121/startup/tt_tt1121 to /etc/init.d

Successfully installed the following scripts :
/etc/init.d/tt_tt1121
/etc/rc.d/rc0.d/K45tt_tt1121
/etc/rc.d/rc1.d/K45tt_tt1121
/etc/rc.d/rc2.d/S90tt_tt1121
/etc/rc.d/rc3.d/S90tt_tt1121
/etc/rc.d/rc5.d/S90tt_tt1121
/etc/rc.d/rc6.d/K45tt_tt1121

在timesten，lsmpusr用户 home目录下，修改.bash_profile
增加：
TIMESTEN_HOME=/opt/TimesTen/tt1121;export TIMESTEN_HOME
PATH=$PATH:$TIMESTEN_HOME/bin
export PATH
LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$TIMESTEN_HOME/lib
export LD_LIBRARY_PATH

卸载
/opt/TimesTen/tt1121/bin/setup.sh -uninstall
主timesten配置
vi /opt/TimesTen/tt1121/info/sys.odbc.ini
增加
[lsmpdb]
Driver=/opt/TimesTen/tt1121/lib/libtten.so
DataStore=/opt/TimesTen/tt1121/info/lsmp/lsmpdb
PermSize=2048 【MB】
TempSize=512 【MB】
PLSQL=1
DatabaseCharacterSet=UTF8
ConnectionCharacterSet=UTF8
#OracleNetServiceName=MyOrclDB

在[ODBC Data Sources] section增加
lsmpdb=TimesTen 11.2.1 Driver
                                                                           
[timesten@localhost info]$ ttdestroy lsmpdb
[timesten@localhost info]$ ttisql lsmpdb

Copyright (c) 1996-2010, Oracle.  All rights reserved.
Type ? or "help" for help, type "exit" to quit ttIsql.



connect "DSN=lsmpdb";
Connection successful: DSN=lsmpdb;UID=timesten;DataStore=/opt/TimesTen/tt1121/info/lsmp/lsmpdb;DatabaseCharacterSet=UTF8;ConnectionCharacterSet=UTF8;DRIVER=/opt/TimesTen/tt1121/lib/libtten.so;PermSize=2048;TempSize=512;TypeMode=0;
(Default setting AutoCommit=1)
Command> quit
Disconnecting...
Done.

Ttisql lsmpdb
create user lsmpusr identified by lsmpusr;
grant create session,create any table,select any table to lsmpusr;
grant delete any table to lsmpusr;
grant create any sequence,drop any sequence,select any sequence to lsmpusr;

ttisql -connStr "UID=lsmpusr;pwd=lsmpusr;dsn=lsmpdb"

Timesten client配置
修改sys.ttconnect.ini
增加
[tt_lsmpdb]
Description=TimesTen Server
Network_Address=10.1.7.4 【对应Timesten server的ip地址】
TCP_PORT=53389 【对应Timesten server时的端口】

vi /opt/TimesTen/tt1121/info/sys.odbc.ini
增加
[lsmpdbCS]
TTC_SERVER= tt_lsmpdb
TTC_SERVER_DSN=lsmpdb
UID=lsmpusr
PWD=lsmpusr

使用ttisqlcs lsmpdbCs测试
修改ubms-jpa
<properties>
                        <property name="hibernate.show_sql" value="ture" />
                        <property name="hibernate.format_sql" value="ture" />
                        <property name="hibernate.connection.driver_class" value="com.timesten.jdbc.TimesTenDriver" />
                        <property name="hibernate.connection.url" value="jdbc:timesten:client:dsn=lsmpdbCS"></property>
                        <property name="hibernate.connection.username" value="lsmpusr"></property>
                        <property name="hibernate.connection.password" value="lsmpusr"></property>
                        <property name="hibernate.dialect" value="org.hibernate.dialect.TimesTenDialect"></property>

                </properties>
修改连接数据库xml文件
  <?xml version="1.0" encoding="UTF-8" standalone="no" ?> 
- <datasources>
- <xa-datasource>
  <jndi-name>LSMPDS</jndi-name> 
  <xa-datasource-class>com.timesten.jdbc.xa.TimesTenXADataSource</xa-datasource-class> 
  <xa-datasource-property name="Url">jdbc:timesten:client:dsn=lsmpdbCS</xa-datasource-property> 
  <driver-class>com.timesten.jdbc.TimesTenDriver</driver-class> 
  <user-name /> 
  <password /> 
  <transaction-isolation>TRANSACTION_READ_COMMITTED</transaction-isolation> 
- <!-- pooling parameters
  --> 
  <min-pool-size>5</min-pool-size> 
  <max-pool-size>100</max-pool-size> 
  <blocking-timeout-millis>5000</blocking-timeout-millis> 
  <idle-timeout-minutes>15</idle-timeout-minutes> 
  <prepared-statement-cache-size>32</prepared-statement-cache-size> 
- <!--                         This is required by TimesTen XA data sources. If it is not included then XA transactions can fail with various
                        transaction management errors including javax.transaction.xa.XAException: errorCode=XAER_PROTO
                
  --> 
  <track-connection-by-tx /> 
- <!--                         sql to call when connection is created or validated <new-connection-sql>SELECT * FROM SYS.TABLES</new-connection-sql>
                        <check-valid-connection-sql>SELECT * FROM SYS.TABLES</check-valid-connection-sql>
                
  --> 
- <!--                  corresponding type-mapping in the standardjbosscmp-jdbc.xml 
                
  --> 
- <metadata>
  <type-mapping>TimesTen</type-mapping> 
  </metadata>
  </xa-datasource>
  </datasources>
Lib下面添加连接timesten的jar包