.环境准备 centos7  iso最小化安装(CentOS-7-x86_64-DVD-1511.iso)，需要在bios打开cpu虚拟化支持

2.第三方源

sudo curl --output /etc/yum.repos.d/fedora-virt-preview.repo https://fedorapeople.org/groups/virt/virt-preview/fedora-virt-preview.repo

yum update

yum -y install epel-release
3.升级内核

[root@kvm21 ~]# uname -a
Linux kvm21 3.10.0-327.el7.x86_64 #1 SMP Thu Nov 19 22:10:57 UTC 2015 x86_64 x86_64 x86_64 GNU/Linux

yum install -y http://www.elrepo.org/elrepo-release-7.0-2.el7.elrepo.noarch.rpm //添加源

yum --enablerepo=elrepo-kernel install -y kernel-ml //安装当前最新内核，以后升级内核直接运行这句就可

yum -y install grub2

# vi /etc/default/grub

修改成 GRUB_DEFAULT=0
grub2-mkconfig -o /boot/grub2/grub.cfg

reboot

yum autoremove kernel-3.10.0-327.13.1.el7.x86_64 //删掉旧内核版本



4.ovs安装

wget http://openvswitch.org/releases/openvswitch-2.7.2.tar.gz

./configure --prefix='/opt/tools/openvswitch'
yum -y install epel-release
yum -y install perl gcc python-pip

pip install six

export PATH=$PATH:/opt/tools/openvswitch/bin:/opt/tools/openvswitch/share/openvswitch/scripts

ovs-ctl start

5 kvm安装

curl http://retspen.github.io/libvirt-bootstrap.sh | sudo sh
6 vnc安装

yum -y groupinstall "X Window System"

yum -y install gnome-classic-session gnome-terminal nautilus-open-terminal control-center liberation-mono-fonts


unlink /etc/systemd/system/default.target


ln -sf /lib/systemd/system/graphical.target /etc/systemd/system/default.target


reboot



yum install tigervnc-server -y


cp /lib/systemd/system/vncserver@.service /etc/systemd/system/vncserver@:1.service

vi /etc/systemd/system/vncserver@:1.service


ExecStart=/sbin/runuser -l root -c "/usr/bin/vncserver %i"
PIDFile=/root/.vnc/%H%i.pid


systemctl daemon-reload


vncpasswd

firewall-cmd --permanent --add-service vnc-server --permanent

firewall-cmd --permanent --add-service http --permanent

systemctl restart firewalld.service

systemctl enable vncserver@:1.service

systemctl start vncserver@:1.service

7 kvm inside kvm

1.修改kvm参数

vi /etc/modprobe.d/kvm-nested.conf

# create new
options kvm_intel nested=1 // 增加如果不存在

reboot

[root@dlp ~]# cat /sys/module/kvm_intel/parameters/nested //查看cpu支持不支持
Y# just enabled

2.如果cpu不支持虚拟化，需要手动虚拟机修改cpu参数
<cpu mode='host-passthrough'>



8.测试环境搭建



测试架构

kvm ovs 安装 - 龙二 - xiangfei_2011的博客




目前是在192.168.0.85安装kvm ，然后启动3台虚拟机 webvirtmgr （192.168.100.91），kvm1（192.168.100.21），kvm2（192.168.100.40）



192.168.0.85

安装kvm ， vnc ， webvirtmgr



192.168.100.91

安装 webvirtmgr

service firewalld stop



kvm1，kvm2

安装kvm ，ovs

kvm(192.168.100.40) 节点配置

hostnamectl set-hostname kvm40

ovs-vsctl add-br br0

ifconfig br0 172.18.0.1/16 up

ovs-vsctl add-port br0 vx1 -- set interface vx1 type=vxlan options:remote_ip=192.168.100.21

firewall-cmd --add-port=4789/tcp --add-port 4789/udp --permanent

firewall-cmd --permanent --add-service vnc-server --permanent

firewall-cmd --reload

ip route add 172.17.0.1/16 dev br0

kvm(192.168.100.21) 节点配置

hostnamectl set-hostname kvm21

ovs-vsctl add-br br0

ifconfig br0 172.17.0.1/16 up

ovs-vsctl add-port br0 vx1 -- set interface vx1 type=vxlan options:remote_ip=192.168.100.40

firewall-cmd --add-port=4789/tcp --add-port 4789/udp --permanent

firewall-cmd --permanent --add-service vnc-server  --permanent

firewall-cmd --reload

ip route add 172.18.0.1/16 dev br0





9 测试结果

1.如果ovs可以vxlan通信，服务器可以跨主机迁移， 热迁移ip不会改变。

2. 重启 libvirtd 虚拟机ip没有影响

3.将hypervisor ip改变，只要修改ovs vxlan 配置，应用内部ip没有影响

10常见问题


































11 遗留问题

1.本次没有搭建共享存储，需要以后测试

2.如果使用同一个网段，ip route 是不是不需要增加，需要测试确定

3.本次使用ovs 测试，linux bridge测试结果以后补上。

4.目前只是在2层网络测试，3层网络需要测试确定

5.ovs dpdk 需要以后测试。

12 ovs-dhcp 配置

server 192.168.100.85

firewall-cmd --add-service=dns --permanent 


firewall-cmd --add-service=dhcp --permanent 

 

firewall-cmd --reload 
ovs-vsctl add-port br0 ovsdhcp -- set interface ovsdhcp type=internal
ifconfig ovsdhcp 172.17.0.4/16 up

dnsmasq --conf-file=ovsdns.conf




[root@libvirt22 dnsmasq]# cat ovsdns.conf
##WARNING: THIS IS AN AUTO-GENERATED FILE. CHANGES TO IT ARE LIKELY TO BE
##OVERWRITTEN AND LOST. Changes to this configuration should be made using:
## virsh net-edit default
## or other application using the libvirt API.
##
## dnsmasq conf file created by libvirt
strict-order
log-queries
log-facility=/tmp/dnsmasq.log
no-resolv
no-hosts
server=172.17.0.4
leasefile-ro
pid-file=/var/run/libvirt/network/ovsdhcp.pid
except-interface=lo
bind-interfaces
interface=ovsdhcp
dhcp-range=172.17.0.4,172.17.254.254,255.255.0.0,86400s
dhcp-no-override
dhcp-lease-max=253
dhcp-hostsfile=/var/lib/libvirt/dnsmasq/ovsdhcp.hostsfile
addn-hosts=/var/lib/libvirt/dnsmasq/ovsdhcp.addnhosts
#log-queries
#log-facility=/tmp/dnsmasq.log




虚拟机启动，需要关掉防火墙，要不然dhcpclient 获取不到



