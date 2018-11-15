# 准备 #
## centos 7 master ##

1. Disable SElinux in CentOS/RHEL 7
/etc/selinux/config  
SELINUX=disabled  

2. Add OpenNebula Repositories
# cat << EOT > /etc/yum.repos.d/opennebula.repo
[opennebula]
name=opennebula
baseurl=https://downloads.opennebula.org/repo/5.6/CentOS/7/x86_64
enabled=1
gpgkey=https://downloads.opennebula.org/repo/repo.key
gpgcheck=1
#repo_gpgcheck=1
EOT

3. Installing the Software
yum install epel-release  
yum install opennebula-server opennebula-sunstone opennebula-ruby opennebula-gate opennebula-flow  

4. Ruby Runtime Installation

/usr/share/one/install_gems  

5. Starting OpenNebula

echo "oneadmin:mypassword" > ~/.one/one_auth
systemctl start opennebula  
systemctl start opennebula-sunstone  