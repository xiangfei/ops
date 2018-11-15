1. Add OpenNebula Repositories

```
# cat << EOT > /etc/yum.repos.d/opennebula.repo
[opennebula]
name=opennebula
baseurl=https://downloads.opennebula.org/repo/5.6/CentOS/7/x86_64
enabled=1
gpgkey=https://downloads.opennebula.org/repo/repo.key
gpgcheck=1
#repo_gpgcheck=1
EOT
```
2. Installing the Software

sudo yum install opennebula-node-kvm
sudo systemctl restart libvirtd

> You may benefit from using the more recent and feature-rich enterprise QEMU/KVM release. The differences between the base (qemu-kvm) and enterprise (qemu-kvm-rhev on RHEL or qemu-kvm-ev on CentOS) packages are described on the Red Hat Customer Portal.

> On CentOS 7, the enterprise packages are part of the separate repository. To replace the base packages, follow these steps:

sudo yum install centos-release-qemu-ev
sudo yum install qemu-kvm-ev

3. Disable SElinux in CentOS/RHEL 7

> SELINUX=disabled

4. Configure Passwordless SSH
```ssh-keyscan <frontend> <node1> <node2> <node3> ... >> /var/lib/one/.ssh/known_hosts```

ssh-copy-id oneadmin@node:/var/lib/one/

> 如果 port不一样，需要定义在ssh 写默认端口和用户

5. Networking Configuration
  * vxlan 802.1q不支持网络隔离 ， 802.1q需要内核支持
  * bridge 简单
  * ovs 支持多租户网络隔离，需要升级内核，流表备份，

6. Storage Configuration
  * Filesystem, to store images in a file form
  	- ssh copy
  * LVM, to store images in LVM logical volumes.
	- 和openstack lvm存储类似，需要san交换机，镜像共享
  * Ceph, to store images using Ceph block devices.
    - kvm ceph 集成
  * Raw Device Mapping, to direct attach to the virtual machine existing block devices in the nodes.
  	- 未测试
  * iSCSI - Libvirt Datastore, to access iSCSI devices through the buil-in qemu support
  	- 未测试
  > 迁移需要2个机器password less 
  > 热迁移需要共享存储

7. Adding a Host to OpenNebula

  onehost create <node01> -i kvm -v kvm

8. Import Currently Running VMs  

  用来导入不是opennebula创建的镜像，不能以one-xx命名