Creating a rootfs image from scratch
In case you want to create the rootfs from scratch you have to

1. Create virtual server on RHEL as you usually do.

Start up the server and do all customisation to the OS. i.e. update OS and add software and configure NI, etc
Create disk for new virtual machine

qemu-img create -f raw myimagerhel62b64.img 2G
Create virtual machine and launch it mount install iso to it

# virt-install --name rhel60b64v3 --ram 1024 --os-type linux -c  /home/timo/rhel-server-6.0-x86_64-dvd.iso --disk  path=/home/timo/rhel60b64v3.img,bus=virtio --hvm --vnc
Check out the the allocated VNC port and connect to the virtual machine by VNC. If VNC ports are not open (e.g. firewall in between your workstation and the bundling server), you can try to tunnel VNC traffic over SSH.

# ps -ef | grep vnc 
qemu     25315     1 99  2012 ?        110-06:01:56 /usr/libexec/qemu-kvm -S -M rhel6.2.0 -enable-kvm -m 1024 -smp 1,sockets=1,cores=1,threads=1 -name rhel62b64v1 -uuid bdfa69ea-ed9c-d0ad-69c4-fd719cf976ea -nodefconfig -nodefaults -chardev socket,id=charmonitor,path=/var/lib/libvirt/qemu/rhel62b64v1.monitor,server,nowait -mon chardev=charmonitor,id=monitor,mode=control -rtc base=utc -no-shutdown -drive file=/home/miikka/rhel62b64v1.img,if=none,id=drive-virtio-disk0,format=raw,cache=none -device virtio-blk-pci,bus=pci.0,addr=0x4,drive=drive-virtio-disk0,id=virtio-disk0,bootindex=2 -drive file=/home/miikka/esclok21-6.2-x86_64.iso,if=none,media=cdrom,id=drive-ide0-1-0,readonly=on,format=raw -device ide-drive,bus=ide.1,unit=0,drive=drive-ide0-1-0,id=ide0-1-0,bootindex=1 -netdev tap,fd=30,id=hostnet0 -device rtl8139,netdev=hostnet0,id=net0,mac=52:54:00:8f:66:89,bus=pci.0,addr=0x3 -chardev pty,id=charserial0 -device isa-serial,chardev=charserial0,id=serial0 -usb -vnc 127.0.0.1:1 -vga cirrus -device virtio-balloon-pci,id=balloon0,bus=pci.0,addr=0x5
Here 127.0.0.1:1 maps to port 5901 (0 -> 5900, 1 -> 5901, 2 -> 5902 etc)

2. Add these lines to /etc/rc.local to get the user ssh keys from Eucalyptus metadata server

# load pci hotplug for dynamic disk attach in KVM (for EBS)
depmod -a
# acpiphp not needed in RHEL6
modprobe acpiphp
# get the user ssh key using the meta-data service
mkdir -p /root/.ssh
echo >> /root/.ssh/authorized_keys
curl --retry 3 --retry-delay 10 -m 45 -s [http://169.254.169.254/latest/meta-data/public-keys/0/openssh-key] | grep 'ssh-rsa' >> /root/.ssh/authorized_keys
echo "AUTHORIZED_KEYS:"
echo "************************"
cat /root/.ssh/authorized_keys
echo "************************"
echo "adding search to resolv.conf"
echo "search emea.nsn-net.net china.nsn-net.net apac.nsn-net.net americas.nsn-net.net" >> /etc/resolv.conf
If you want to use Userdata, see the section below 

3. Make sure that NTP is running on virtual server. Also see Clock settings for instances if you want Linux to treat hardware clock properly after boot-up.

4. Remove network identity of the server and disable IPv6. Zeroconf needs to also be disabled because it causes problems with userdata.

vi /etc/sysconfig/network
NETWORKING=yes
NETWORKING_IPV6=no
#if you need NIS
NISDOMAIN=eelinnis.emea.nsn-net.net
#Disable Zeroconf
NOZEROCONF=yes
vi /etc/hosts
Remove all custom host information

vi /etc/sysconfig/network-scripts/ifcfg-eth0
DEVICE=eth0
BOOTPROTO=dhcp
ONBOOT=yes
TYPE=Ethernet
Networking should be set to DHCP as the cloud is responsible for providing IPs to started instances via DHCP.

Remove /etc/udev.rules/persistent-net rules file if existing. In newer OS:es you might need to do wipefs also.

For Redhat 6 the file you need to clear is /etc/udev/rules.d/70-persistent-net.rules

5. Modify /etc/fstab to look like this

/dev/vda1               /                       ext3    defaults        0 0
tmpfs                   /dev/shm                tmpfs   defaults        0 0
devpts                  /dev/pts                devpts  gid=5,mode=620  0 0
sysfs                   /sys                    sysfs   defaults        0 0
proc                    /proc                   proc    defaults        0 0
/dev/vda2               /ephemeral              ext2    defaults        0 0
/dev/vda3               swap                    swap    defaults        0 0
6. Copy the /boot/vmlinuz & initrd to client server for uploading to euca (if custom kernel/ramdisk are needed and hasn't already been registered to the cloud)

scp /boot/initrd... root@client:/tmp
scp /boot/vmlinuz... root@client:/tmp
7. Copy the virtul server file to working directory

cp /var/lib/libvirt/images/<yourimage>.img /tmp

The we need to extract out the raw partition data for the rootfs, as the virtual disk image contains the full disk (not just the root partition):

