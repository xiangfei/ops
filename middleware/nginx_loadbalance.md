user  nginx;  
worker_processes  2;  
  
error_log   /var/log/nginx/error.log;  
#error_log  /var/log/nginx/error.log  notice;  
#error_log  /var/log/nginx/error.log  info;  
  
pid         /var/run/nginx.pid;  
  
  
events {  
    use epoll; # 采用epoll,是linux下最快的event  
    worker_connections  2048;  
}  
  
  
http {  
    include       mime.types;  
    default_type  application/octet-stream;  
    sendfile        on;  
    keepalive_timeout  65;  
    #gzip  on;  
  
    #反向代理配置 ,向内网6台jboss转发  
    upstream jboss5 {  
        server 172.16.201.127:8080 weight=10;  
        server 172.16.201.128:8080 weight=10;  
        server 172.16.201.129:8080 weight=10;  
        server 172.16.201.130:8080 weight=10;  
        server 172.16.201.131:8080 weight=10;  
        server 172.16.201.132:8080 weight=10;  
    }  
  
    server {  
        listen       80;  
        server_name  localhost;  
  
        location ~ ^/nginx_status/ {  
            stub_status on;  
            access_log off;  
        }  
  
        location / {  
            proxy_pass http://jboss5;  
        }  
  
        error_page   500 502 503 504  /50x.html;  
        location = /50x.html {  
            root   html;  
        }  
  
    }  
  
}  