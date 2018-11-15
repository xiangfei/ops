GRUB_TIMEOUT=5
GRUB_DEFAULT=saved
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="rd.lvm.lv=centos/root rd.lvm.lv=centos/swap crashkernel=auto rhgb quiet console=ttyS0"
GRUB_DISABLE_RECOVERY="true"

stty -F /dev/ttyS0 speed 9600
grub2-mkconfig -o /boot/grub2/grub.cfg
systemctl start getty@ttyS0
needs send enter button 
