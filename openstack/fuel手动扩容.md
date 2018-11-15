系统为ubuntu，lvm模式 系统创建3个vgs：cinder–900G 、 os—400G 、 vm—500G os下lvs有 root—200G（ext4）；swap—-290G ；boot–200MB vm下lvs有nova—-500G（xfs）

1. 存储网络

br-storage

root@node109:/etc/network/interfaces.d# ip a | grep br-storage 3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq master br-storage state UP group default qlen 1000 23: br-storage: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000  

inet 192.168.216.22/24 brd 192.168.216.255 scope global br-storage

2. 管理网络

br-mgmt
root@node109:/etc/network/interfaces.d# ip a | grep br-mgmt 17: eth0.218@eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master br-mgmt state UP group default qlen 1000 18: br-mgmt: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000

inet 192.168.218.22/24 brd 192.168.218.255 scope global br-mgmt


3. 业务网络

br-aux

root@node109:/etc/network/interfaces.d# ip a | grep br-aux 4: eth3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq master br-aux state UP group default qlen 1000 22: p_e52381cd-0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 65000 qdisc noqueue master br-aux state UNKNOWN group default qlen 1 41: br-aux: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000

note ! br-aux 需要add 物理网卡，否则无法跨机 , 不需要设置ip，需要确保物理网卡状态UP

4. pxe网络

br-fw-admin	10.20.1.x

root@node109:/etc/network/interfaces.d# ip a | grep br-fw-admin 2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq master br-fw-admin state UP group default qlen 1000 16: br-fw-admin: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000

  inet 10.20.1.109/24 brd 10.20.1.255 scope global br-fw-admin

安装步骤
1. 在计算节点copy /etc/network/interfaces , inerface.d 目录，

2. 替换 apt源

mv /etc/apt /etc/apt.bak

scp -r root@10.20.1.109:/etc/apt /etc

apt-get update

3. 配置ntp

4. 安装系统工具

apt-get install cinder-backup

apt-get install cinder-volume

apt-get install neutron-openvswitch-agent

apt-get install nova-compute

1 .locale: Cannot set LC_ALL to default locale: No such file or directory # ingore 2. 配置全部跳过 ,后续手动配置

5. 修改存储

配置

修改网桥，需要ovs安装成功才能重启

cinder

cd /etc mv cinder cinder.bak

scp root@10.20.1.109:/etc/cincder .

chown cinder:cinder /etc/cinder/api-paste.ini chown cinder:cinder /etc/cinder/logging.conf chmod a+x /etc/cinder 修改/etc/cinder/cinder.conf

iscsi_ip_address=192.168.216.23 #192.168.216.3 本地存储地址 host = node110 storage_availability_zone = newnova default_availability_zone = newnova

nova

cd /etc mv nova nova.bak scp root@10.20.1.109:/etc/nova .

chown nova:nova /etc/nova/api-paste.ini chown nova:nova /etc/nova/logging.conf chmod a+x /etc/nova

修改 /etc/nova/nova.conf

vncserver_proxyclient_address=192.168.218.23 my_ip=192.168.218.23 #192.168.216.3 本地管理地址

neutron cd /etc mv neutron neutron.bak scp root@10.20.1.109:/etc/neutron . chown neutron:neutron /etc/neutron/api-paste.ini chown neutron:neutron /etc/neutron/neutron.conf chown neutron:neutron /etc/neutron/plugins/ml2/openvswitch_agent.ini chmod a+x /etc/neutron 修改 /etc/neutron/neutron.conf bind_host = 192.168.218.23

root@node110:/etc/neutron# service cinder-backup restart cinder-backup stop/waiting cinder-backup start/running, process 56271 root@node110:/etc/neutron# root@node110:/etc/neutron# service cinder-volume restart

root@node110:/etc/neutron# service nova-compute restart nova-compute stop/waiting nova-compute start/running, process 56457

root@node110:/etc/neutron# service neutron-openvswitch-agent restart neutron-openvswitch-agent stop/waiting neutron-openvswitch-agent start/running, process 56903

2018-08-21 10:55:41.373 56903 ERROR neutron.agent.linux.async_process [-] Error received from [ovsdb-client monitor Interface name,ofport,external_ids –format=json]: 2018-08-21T02:55:41Z|00001|fatal_signal|WARN|terminating with signal 15 (Terminated) #ignore

#GRUB_TIMEOUT=1 GRUB_DISTRIBUTOR=“$(sed 's, release .*$,,g' /etc/system-release)” #GRUB_DEFAULT=saved #GRUB_DISABLE_SUBMENU=true GRUB_TERMINAL=“serial console” GRUB_SERIAL_COMMAND=“serial –speed=115200 –unit=0 –word=8 –parity=no –stop=1” #GRUB_CMDLINE_LINUX=“console=tty0 crashkernel=auto console=ttyS0,115200” #GRUB_SERIAL_COMMAND=“serial –speed=38400 –unit=0 –word=8 –parity=no –stop=1”

#GRUB_DISABLE_RECOVERY=“true”

console connect

GRUB_TIMEOUT=5 GRUB_DEFAULT=saved GRUB_DISABLE_SUBMENU=true GRUB_TERMINAL_OUTPUT=“console” GRUB_CMDLINE_LINUX=“crashkernel=auto rhgb quiet console=ttyS1,115200 console=tty0 console=tty1 console=ttyS0,115200” GRUB_DISABLE_RECOVERY=“true”

grub2-mkconfig -o /boot/grub2/grub.cfg

reboot