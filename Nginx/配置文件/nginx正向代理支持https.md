#环境说明：
正向代理的nginx安装正常安装就可以，没有特别的要求
#说明：
nginx当正向代理的时候，通过代理访问https的网站会失败，而失败的原因是客户端同nginx代理服务器之间建立连接失败，并非nginx不能将https的请求转发出去。因此要解决的问题就是客户端如何同nginx代理服务器之间建立起连接。有了这个思路之后，就可以很简单的解决问题。我们可以配置两个SERVER节点，一个处理HTTP转发，另一个处理HTTPS转发，而客户端都通过HTTP来访问代理，通过访问代理不同的端口，来区分HTTP和HTTPS请求。
````
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

include /usr/share/nginx/modules/*.conf;

events {
worker_connections 1024;
}

http {
log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
'$status $body_bytes_sent "$http_referer" '
'"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    include /etc/nginx/conf.d/*.conf;

#HTTP proxy       #这里位http的正向代理配置
    server{
        resolver 8.8.8.8;
        access_log /var/log/nginx/access_proxy-80.log main;
    listen 80;
    location / {
    root html;
    index index.html index.htm;
    proxy_pass $scheme://$host$request_uri;
    proxy_set_header HOST $http_host;
    proxy_buffers 256 4k;
    proxy_max_temp_file_size 0k;
    proxy_connect_timeout 30;
    proxy_send_timeout 60;
    proxy_read_timeout 60;
    proxy_next_upstream error timeout invalid_header http_502;
    }
    error_page 500 502 503 504 /50x.html;
    location = /50x.html {
    root html;
        }
    }

#HTTPS proxy        #这里为：https的正向代理配置
    server{
    resolver 8.8.8.8;
    access_log /var/log/nginx/access_proxy-443.log main;
    listen 443;
    location / {
    root html;
    index index.html index.htm;
    proxy_pass https://$host$request_uri;
    proxy_buffers 256 4k;
    proxy_max_temp_file_size 0k;
    proxy_connect_timeout 30;
    proxy_send_timeout 60;
    proxy_read_timeout 60;
    proxy_next_upstream error timeout invalid_header http_502;
    }
    error_page 500 502 503 504 /50x.html;
    location = /50x.html {
    root html;
    }
    }
}
````
#测试
1、如果访问HTTP网站，可以直接这样的方式: curl --proxy proxy_server-ip:80
2、如果访问HTTPS网站，例如https://www.alipay.com，那么可以使用nginx的HTTPS转发的server：
curl --proxy proxy_server:443 http://www.alipay.com
3、使用浏览器访问，在浏览器的设置里设置代理服务器

参考：https://www.cnblogs.com/hei-ma/p/9722478.html