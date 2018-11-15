# opennebula 需要手动配置 #
1. 安装ovs
 yum install centos-release-openstack-rocky
 yum install openvswitch
2. ovs网桥,增加开机自动启动
  - eth0 配置
	```DEVICE=eth0
	ONBOOT=yes
	DEVICETYPE=ovs
	TYPE=OVSPort
	OVS_BRIDGE=ovsbridge0
	BOOTPROTO=none
	HOTPLUG=no```
  - gre  配置
	```DEVICE=ovs-gre0
	ONBOOT=yes
	DEVICETYPE=ovs
	TYPE=OVSTunnel
	OVS_BRIDGE=ovsbridge0
	OVS_TUNNEL_TYPE=gre
	OVS_TUNNEL_OPTIONS="options:remote_ip=A.B.C.D"
	```
  - vxlan 配置
	```DEVICE=ovs-gre0
	ONBOOT=yes
	DEVICETYPE=ovs
	TYPE=OVSTunnel
	OVS_BRIDGE=ovsbridge0
	OVS_TUNNEL_TYPE=vxlan
	OVS_TUNNEL_OPTIONS="options:remote_ip=A.B.C.D"```
  - patch  配置
	```DEVICE=patch-ovs-0
	ONBOOT=yes
	DEVICETYPE=ovs
	TYPE=OVSPatchPort
	OVS_BRIDGE=ovsbridge0
	OVS_PATCH_PEER=patch-ovs-1```
  - bound 配置
	```DEVICE=bond0
	ONBOOT=yes
	DEVICETYPE=ovs
	TYPE=OVSBond
	OVS_BRIDGE=ovsbridge0
	BOOTPROTO=none
	BOND_IFACES="gige-1b-0 gige-1b-1 gige-21-0 gige-21-1"
	OVS_OPTIONS="bond_mode=balance-tcp lacp=active"
	HOTPLUG=no```
  - tag 配置
	```DEVICE=vlan100
	ONBOOT=yes
	DEVICETYPE=ovs
	TYPE=OVSIntPort
	BOOTPROTO=static
	IPADDR=A.B.C.D
	NETMASK=X.Y.Z.0
	OVS_BRIDGE=ovsbridge0
	OVS_OPTIONS="tag=100"
	OVS_EXTRA="set Interface $DEVICE external-ids:iface-id=$(hostname -s)-$DEVICE-vif"
	HOTPLUG=no```
  - common
	```DEVICE=intbr0
	ONBOOT=yes
	DEVICETYPE=ovs
	TYPE=OVSIntPort
	OVS_BRIDGE=ovsbridge0
	BOOTPROTO=static
	IPADDR=A.B.C.D
	NETMASK=X.Y.Z.0
	HOTPLUG=no```


> 需要内核支持ovs,最好升级到最新的内核
> vxlan gre ,需要打开端口
> 临时配置
>> ovs-vsctl add-br ovs01
>> ovs-vsctl add-port ovs01 eth2 tag=100
>> ovs-vsctl add-port ovs01 vxlan0 tag=100 -- set interface vxlan0 type=vxlan options:key=100 options:remote_ip=172.16.100.2
>> ovs-vsctl add-port ovs01 vxlan0 tag=100 -- set interface vxlan0 type=gre options:key=100 options:remote_ip=172.16.100.2
>> ovs-vsctl add-port ovs01 vxlan0 tag=100 -- set interface type=internal

