    




  下载LOFTER我的照片书  |
tools  cisco packet tracer
prepare
1. move 2 layer switcher,  machine to the dashboard.
2.config the  machine ip , mac address
3. in 2 layer switcher config vlan , in vlan config the ip  

scripts demo 
configure t 
vlan 10
hostname test
end
wr

# assgin ip for vlan
configure t
interface vlan 10
ip address 192.168.1.11 255.255.255.0
no shutdown
end
wr
# assgin  port for vlan
configure t
interface range FastEthernet0/10-12
switch access  vlan 10
end
wr

#set port  mode 
configure t
interface FastEthernet0/10
switch mode  access
end
wr

# interface binding 
configure t
interface range  f0/12-14
channel-group 1 mode on
wr