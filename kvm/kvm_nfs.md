nfs install 
yum -y install nfs-utils

[root@localhost ~]# firewall-cmd  --add-service=nfs --permanent
success
[root@localhost ~]# firewall-cmd  --reload

[root@localhost ~]# cat /etc/exports
/root *(rw,no_root_squash)