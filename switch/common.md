命名规则
交换机命名通常是  型号+类型   configure t    hostname   xxxx
vlan命名规则  交换机+连接交换机 configure t   vlan 201  name  xxxx
vlan描述  交换机+连接交换机 configure t    interface  vlan 201  description  xxxx

端口类型 trunk (交换机+ 交换机)，  access （交换机 + server） ， 自动适配


vlan 划分
server通常连接2层交换机（使用直连线copper straight），通常在3层交换机给vlan 划分ip address ，2层不需要做
 步骤
1.修改端口类型 ， configure t  interface  Fa0/1 swithport mode access  description  yyy
2.创建vlan  configure t  vlan 201   name switch+ switch  description yyy
3.将端口划分给vlan  configure t  interface  Fa0/1 swithport  access  vlan 201
网线连接

2层交换机连接3层交换机使用直连线copper straight）
步骤
修改2层交换机端口 configure t  interface  Fa0/2 swithport mode trunk
修改3层交换机端口 configure t  interface  Fa0/2 swithport mode trunk
网线连接 Fa0/2 
在2层交换机划创建vlan2 ，分配端口
在3层交换机划分vlan2  ，  写网关  ip address  configure t  vlan 2  ip address 192.168.1.254  255.255.255.0
（在2层交换机和3层交换机的vlan必须一样，否则失败）