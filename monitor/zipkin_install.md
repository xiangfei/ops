# 安装 #



# 配置 #

1. storage mysql
  - zipkin/zipkin-storage/mysql-v1/src/main/resources/mysql.sql
  - ```
java -server -jar zipkin-server-2.11.8-exec.jar --zipkin.storage.type=mysql --zipkin.storage.mysql.host=localhost --zipkin.storage.mysql.port=3306 --zipkin.storage.mysql.username=root --zipkin.storage.mysql.password=123456 --zipkin.storage.mysql.db=zipkin
  ```