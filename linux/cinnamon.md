yum -y install epel-release

yum groupinstall "Server with GUI" -y # yum groupinstall "X Window System"

yum install cinnamon gdm liberation* gnome-terminal gnome-icon-theme-legacy.noarch

echo "exec /usr/bin/cinnamon-session" >> ~/.xinitrc

ln -sf /lib/systemd/system/runlevel5.target /etc/systemd/system/default.target


yum -y install epel-release

yum groupinstall "X Window System" "Fonts"



yum --enablerepo=epel -y install cinnamon*
yum install  gdm liberation* gnome-terminal

echo "exec /usr/bin/cinnamon-session" >> ~/.xinitrc

startx 

yum install fcitx* //pinying test failed


windows theme https://github.com/B00merang-Project/Windows-10-Dark  theme  unzip to  ~/.themes

https://github.com/Elbullazul/Windows-10/releases/download/v0.9.7/Windows.10.Icons.v0.4.2.zip icons ~/.icons



config keyboards  #

mkdir -p  ~/.fluxbox/styles   ~/.config/tint2

  cp Windows\ 10\ Dark/tint2/tint2rc  ~/.config/tint2/wintint2
  cp -r Windows\ 10\ Dark/fluxbox/Windows\ 10\ Dark/ ~/.fluxbox/styles

换背景图片

运行eclipse失败   export DISPLAY=:0
yum -y install gtk*


aptana shell cannot run

[root@localhost aptana]# chmod u+x ./plugins/com.aptana.terminal_3.0.0.201802081530/os/linux/x86_64/redtty
[root@localhost aptana]# chmod u+x ./plugins/com.aptana.terminal_3.0.0.201802081530/os/linux/x86/redtty

/etc/locale.conf LANG="zh_CN.UTF-8"

yum -y install ibus-libpinyin