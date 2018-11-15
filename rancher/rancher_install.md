1. 安装 
参考官网，docker 1.2.6 
linux centos minal install   ， enable ip ， install openssh , openssh-server , git , net-tools, mysql ,rancher 1.5

docker 安装
curl https://releases.rancher.com/install-docker/1.12.sh | sh
使用 网易的镜像，没有安装docker-proxy ，会导致rancher 启动失败

enable service
systemctl enable docker
systemctl disable firewalld (不能停 需要iptables 流表控制)
/usr/sbin/setenforce 0



;
> CREATE DATABASE IF NOT EXISTS cattle COLLATE = 'utf8_general_ci' CHARACTER SET = 'utf8';
> GRANT ALL ON cattle.* TO 'cattle'@'%' IDENTIFIED BY 'cattle';
> GRANT ALL ON cattle.* TO 'cattle'@'localhost' IDENTIFIED BY 'cattle';

sudo docker run -d --restart=unless-stopped -p 8080:8080 rancher/server \
    --db-host myhost.example.com --db-port 3306 --db-user username --db-pass password --db-name cattle

comments 
docker 启动，会在iptables 增加端口，docker kill 会保留，端口， docker rm 会在iptables 删除端口