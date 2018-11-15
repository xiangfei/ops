ubuntu 系统

apt-get intall kpartx

apt-get install qemu-utils

modprobe nbd max_part=8

qemu-nbd -c /dev/nbd0 /var/lib/one/datastores/0/7/opendaylight.qcow2 

mount /dev/nbd0p1 /mnt/

umount /mnt

qemu-nbd  -d  /dev/nbd0

qemu-kvm-tools  , qemu-img

cat /etc/redhat-release

#-> CentOS Linux release 7.4.1708 (Core)

查看自己内核版本

uname -r       

#-> 3.10.0-693.el7.x86_64


 开始安装编译模块

yum install kernel-devel kernel-headers

cd /tmp

wget http://vault.centos.org/7.4.1708/os/Source/SPackages/kernel-3.10.0-693.el7.src.rpm

rpm -ihv kernel-3.10.0-693.el7.src.rpm  (这时会在/root/rpmbuild/SOURCES下生成tar.xz包)

cd ~/rpmbuild/SOURCES

tar Jxvf linux-3.10.0-693.el7.tar.xz -C/usr/src/kernels/

cd /usr/src/kernels/

mv $(uname -r) $(uname -r)-old

mv linux-3.10.0-693.el7 $(uname -r)

cd $(uname -r)

make mrproper

cp ../$(uname -r)-old/Module.symvers ./

cp /boot/config-$(uname -r) ./.config

make oldconfig

make prepare

make scripts

make CONFIG_BLK_DEV_NBD=m M=drivers/block

cp drivers/block/nbd.ko /lib/modules/$(uname -r)/kernel/drivers/block/

depmod -a


Guestfish

guestfish --rw -a centos63_desktop.img

run

list-filesystems

mount /dev/vg_centosbase/lv_root /

rm /etc/udev/rules.d/70-persistent-net.rules

edit /etc/sysconfig/network-scripts/ifcfg-eth0

exit

Guestmount

guestmount -a centos63_desktop.qcow2 -i --rw /mnt


rpm -qa --dbpath /mnt/var/lib/rpm

umount /mnt