#1

控制节点 vip ip 192.168.218.3(管理网段)  , 192.168.208.3(外部访问) offline  

影响
mysql, openstack(所有组件), haproxy, pacemaker , corosync

未影响 memcache rabbitmq 

解决方式

控制节点
1. 增加 192.168.218.3  

ip address add 192.168.218.3/24 dev br-mgmt

2. 重启mysql

bin/sh /usr/bin/mysqld_safe  --socket=/var/run/mysqld/mysqld.sock --datadir=/var/lib/mysql --user=mysql --wsrep-new-cluster --bind-address=192.168.218.3 --port=3306

3. disable haproxy , corosync, pacemaker

4. 修改keystone 配置文件 /etc/keystone/keystone.conf

#admin_bind_host=192.168.218.6
#public_bind_host=192.168.218.6
admin_bind_host=192.168.218.3
public_bind_host=192.168.218.3

service keystone restart

5. 修改glance 配置文件 /etc/glance/glance-api.conf
bind_host = 0.0.0.0
#bind_host = 192.168.218.3

service  glance-api glance-glare glance-registry  restart

6. 修改nova配置文件 /etc/nova/nova.conf
#novncproxy_base_url=http://192.168.218.6:6080/vnc_auto.html
novncproxy_base_url=http://192.168.208.3:6080/vnc_auto.html

#osapi_compute_listen=192.168.218.6
#osapi_compute_listen=192.168.218.3
osapi_compute_listen=0.0.0.0

#novncproxy_host=192.168.218.6
novncproxy_host=192.168.208.3

service  nova-api nova-conductor  nova-consoleauth nova-novncproxy nova-scheduler nova-cert restart

7. 修改neutron配置文件  /etc/neutron/neutron.conf
bind_host = 0.0.0.0
#bind_host = 192.168.218.6

service  neutron-dhcp-agent neutron-l3-agent neutron-metadata-agent neutron-openvswitch-agent neutron-server restart


8. 修改cinder配置文件  /etc/cinder/cinder.conf

#osapi_volume_listen = 192.168.218.3
osapi_volume_listen = 0.0.0.0

service cinder-api cinder-scheduler cinder-volume cinder-backup restart

9.修改apache2 配置文件 /etc/apache2/ports.conf
Listen 10.20.1.104:8888
Listen 127.0.0.1:80
~#Listen 192.168.218.6:35357
~#Listen 192.168.218.6:5000
Listen 192.168.218.6:80
~#Listen 192.168.218.3:5000
~#Listen 192.168.218.3:35357
Listen 192.168.218.3:80
Listen 192.168.208.3:80
~#Listen 192.168.208.3:8774
~#Listen 192.168.208.3:5000
~#add wangxp 20180727
~#Listen 192.168.208.3:35357
~#Listen 192.168.208.3:8774

10. 修改apache2 配置文件 /etc/apache2/ports.conf

~#<VirtualHost 192.168.218.6:80>
<VirtualHost 192.168.218.3:80>

service apache2 restart


11. iptables 增加规则
iptables -t nat -A OUTPUT -d 192.168.208.3 -j DNAT --to-destination 192.168.218.3

12. nova 

#OPENSTACK_HOST = "127.0.0.1"
#OPENSTACK_KEYSTONE_URL = "http://%s:5000/v2.0" % OPENSTACK_HOST
OPENSTACK_HOST = "192.168.218.3"
OPENSTACK_KEYSTONE_URL = "http://%s:5000/v3" % OPENSTACK_HOST
OPENSTACK_KEYSTONE_DEFAULT_ROLE = "user"

service apache2 restart

13. ip 固定



计算节点

service cinder-volume cinder-backup neutron-openvswitch-agent nova-compute restart
 

ruby /var/tmp/one/