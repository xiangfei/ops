nginx.conf实例
user  lsmpusr lsmpusr;

worker_processes  12;

error_log  logs/error.log  error;

worker_rlimit_nofile 65535;

events {

    worker_connections  102400;

}

 

http {

    include       mime.types;

    default_type  application/octet-stream;

    access_log  off;

    sendfile        on;

    #tcp_nopush     on;

    tcp_nodelay     on;

    keepalive_timeout  65;

    gzip               on;

    gzip_min_length    1k;

    gzip_buffers       16 64k;

    gzip_http_version  1.1;

    gzip_types         text/plain application/x-javascript text/css application/xml;

    gzip_comp_level    9;

    gzip_vary          on;

    client_header_buffer_size 4k;

    client_max_body_size 8m;

    open_file_cache max=102400 inactive=20s;

    fastcgi_connect_timeout 300;

    fastcgi_send_timeout 300;

    fastcgi_read_timeout 300;

    fastcgi_buffer_size 32k;

    fastcgi_buffers 8 32k;

 

    # HTTPS server

    server {

        listen       443;

        server_name  server1;

        location / {

            proxy_pass http://127.0.0.1:9080;

            proxy_set_header  X-Real-IP  $remote_addr;

        }

        ssl                  on;

        ssl_certificate      /usr/local/nginx/conf/cfcanewco.cer;

        ssl_certificate_key  /usr/local/nginx/conf/ca.key;

        ssl_session_timeout  5m;

        ssl_protocols  SSLv2 SSLv3 TLSv1;

        ssl_ciphers  HIGH:!aNULL:!MD5;

        ssl_prefer_server_ciphers   on;

        error_page   500 502 503 504  /50x.html;

        location = /50x.html {

            root   html;

        }

    }

}

1.1.2  nginx.conf字段说明
user  lsmpusr lsmpusr;

非守护进程运行的用户和组

worker_processes  12;

工作进程数，一般与 CPU 核数等同

gzip               on;

该指令用于开启或关闭gzip模块(on/off)

gzip_min_length    1k;

设置允许压缩的页面最小字节数，页面字节数从header头得content-length中进行获取。默认值是0，不管页面多大都压缩。建议设置成大于1k的字节数，小于1k可能会越压越大。

gzip_buffers       16 64k;

设置系统获取几个单位的缓存用于存储gzip的压缩结果数据流。4 16k代表以16k为单位，安装原始数据大小以16k为单位的4倍申请内存。

gzip_http_version  1.1;

识别http的协议版本(1.0/1.1)

gzip_comp_level    9;

gzip压缩比，1压缩比最小处理速度最快，9压缩比最大但处理速度最慢，数值越大传输速度越快，但越消耗cpu。

gzip_types         text/plain application/x-javascript text/css application/xml;

匹配mime类型进行压缩，无论是否指定,”text/html”类型总是会被压缩的。

gzip_vary          on;

和http头有关系，加个vary头，给代理服务器用的，有的浏览器支持压缩，有的不支持，所以避免浪费不支持的也压缩，所以根据客户端的HTTP头来判断，是否需要压缩

listen       443;https

对外的端口号

proxy_pass http://127.0.0.1:9080;

我们的webservice的url

ssl_certificate      /usr/local/nginx/conf/cfcanewco.cer;

颁发机构签发的证书目录。

ssl_certificate_key  /usr/local/nginx/conf/ca.key;

自己私钥的目录。