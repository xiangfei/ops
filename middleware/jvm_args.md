    




  下载LOFTER我的照片书  |
[ $1 != stop ] && JAVA_OPTS="-server -XX:PermSize=128M -XX:MaxPermSize=256M -Xms1024M -Xmx1024M -Xmn384M -XX:SurvivorRatio=8 -XX:+DisableExplicitGC -XX:+UseConcMarkSweepGC -XX:+UseParNewGC -XX:+UseCMSCompactAtFullCollection -XX:CMSFullGCsBeforeCompaction=10 -XX:+CMSClassUnloadingEnabled -XX:+CMSParallelRemarkEnabled -XX:+UseCMSInitiatingOccupancyOnly -XX:CMSInitiatingOccupancyFraction=70 -XX:TargetSurvivorRatio=90 -XX:MaxTenuringThreshold=15 -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.authenticate=true -Dcom.sun.management.jmxremote.password.file=../shared/conf/jmxremote.password -Dcom.sun.management.jmxremote.access.file=../shared/conf/jmxremote.access  -Djava.rmi.server.hostname=${tomcat_jmx_ip} -Dtomcat.working.group=${tomcat_group} -XX:-ReduceInitialCardMarks -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=${heap_bin} -XX:+PrintGCDetails -XX:+PrintGCDateStamps -Xloggc:${gc_log}  -Xdebug -Xrunjdwp:transport=dt_socket,address=9909,server=y,suspend=n"