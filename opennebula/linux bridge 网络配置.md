1. centos 配置bridge网桥
  -  配置网卡
	```vim /etc/sysconfig/network-scripts/ifcfg-enp8s0 
	TYPE=Ethernet
	DEVICE=enp8s0
	NAME=enp8s0
	BOOTPROTO=none
	ONBOOT=yes
	BRIDGE=br0
	```

  - 配置网桥
	```	vim /etc/sysconfig/network-scripts/ifcfg-br0
	TYPE=Bridge
	DEVICE=br0
	BOOTPROTO=static
	ONBOOT=yes
	IPADDR=192.168.1.200
	NETMASK=255.255.255.0
	GATEWAY=192.168.1.1
	DNS1=114.114.114.114
	```

> sunstone 选择网络bridge,DNS,network mask,gateway 现有网络存在,没有使用dnsmasq
> linux 内核不需要升级