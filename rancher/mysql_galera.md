prepare
https://octoperf.com/blog/2016/11/17/rancher-galera-cluster/
Galera Server One available at IP address 1.1.1.1,
Galera Server Two available at IP address 2.2.2.2,
Galera Server Three available at IP address 3.3.3.3,
Rancher available at IP address 4.4.4.4.

1.安装rancher server 
sudo docker run -d -p 8080:8080 rancher/server (不需要restart = true)，通过rancher 安装 mysql galera

2. add hosts to rancher ,add custom host

3.安装galera
通过docker compose 或者 rancher compose 安装 
文件 https://github.com/rancher/catalog-dockerfiles/tree/master/galera/0.2.0

The docker-compose file contains the password used for the MySQL root user and for the cattle database :

de style="box-sizing: border-box; font-family: monospace, monospace; font-size: 1em;"  >MYSQL_ROOT_PASSWORD: "password"
MYSQL_DATABASE: "cattle"
MYSQL_USER: "cattle"
MYSQL_PASSWORD: "cattle
de>
You may want to edit them for a production installation.

In any case you need to edit the Galera load balancer configuration:

expose(可以不替换，rancher server安装在galera  server也行，)

galera-lb:
  ports:
  - 3306:3307/tcp
  labels:
    io.rancher.scheduler.affinity:host_label: main=true
  tty: true
  image: rancher/load-balancer-service
  links:
  - galera:galera
  stdin_open: truede>
de style="box-sizing: border-box; font-family: monospace, monospace; font-size: 1em;"  >
de>
de style="box-sizing: border-box; font-family: monospace, monospace; font-size: 1em;"  >We removed the de style="box-sizing: border-box; font-family: monospace, monospace; white-space: normal;"  >expose: - 3306:3307/tcpde> configuration and replaced it with a public mapping. By doing so our database cluster is publicde>
rancher add stack to create nodes
指定 galery main  server main=true , 只需要指定一台，
导出数据库
Open the Rancher UI (http://4.4.4.4:8080) and go to the Admin > High-Availability page,
click export
或者 wget http://4.4.4.4:8080/v1/haconfigs/haconfig/dbdump
在 galery db cluster 更新数据 mysql -u cattle --password=cattle -h 1.1.1.1 cattle < rancher-mysql-dump.sql
删掉temp server 
启动
de style="box-sizing: border-box; font-family: monospace, monospace; font-size: 1em;"  >docker run -d -p 8080:8080 \
--restart=unless-stopped \
-e CATTLE_DB_CATTLE_MYSQL_HOST=1.1.1.1 \
-e CATTLE_DB_CATTLE_MYSQL_PORT=3306 \
-e CATTLE_DB_CATTLE_MYSQL_NAME=cattle \
-e CATTLE_DB_CATTLE_USERNAME=cattle \
-e CATTLE_DB_CATTLE_PASSWORD=cattle \
-v /var/run/docker.sock:/var/run/docker.sock \
rancher/serverde>

de style="box-sizing: border-box; font-family: monospace, monospace; font-size: 1em;"  >
de>
de style="box-sizing: border-box; font-family: monospace, monospace; font-size: 1em;"  >
de>