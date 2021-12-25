#1.安装tengine启用stream模块
注意nginx版本要在1.9以上
./configure --prefix=/usr/local/nginx --user=www --group=www --with-http_ssl_module --with-http_stub_status_module --with-file-aio --with-stream --with-stream_ssl_preread_module --with-stream_ssl_module && make && make install

/usr/sbin/groupadd -f www
/usr/sbin/useradd -g www www

#2.配置4层代理
--------------------------------------------------------------------------------
````
#内网

worker_processes 1;

events {
worker_connections 1024;
}

stream {
server {
listen 80;
proxy_connect_timeout 1s; #后端链接空闲超时断开
proxy_timeout 10s; #后端连接超时时间
proxy_pass 192.168.3.72:80;
}
server {
listen 443;
proxy_connect_timeout 1s; #后端链接空闲超时断开
proxy_timeout 10s; #后端连接超时时间
proxy_pass 192.168.3.72:443;
}
}

#DMZ
worker_processes 1;

events {
worker_connections 1024;
}

stream {
ssl_preread on;
map_hash_max_size 128;
map_hash_bucket_size 128;
map $ssl_preread_server_name $upstream_name {
isrm-dev-public.obs.cn-east-2.myhuaweicloud.com tcp_proxy_hw_isrm-dev-public;
csrzic.test.isrm.going-link.com tcp_proxy_csrzic;
test.isrm.going-link.com tcp_proxy_csrzic;
# default tcp_proxy_hw_isrm-dev-public;
default tcp_proxy_csrzic;
}
upstream tcp_proxy_csrzic {
hash $remote_addr consistent; #远程地址做个hash
server csrzic.test.isrm.going-link.com:443;
}
upstream tcp_proxy_hw_isrm-dev-public {
hash $remote_addr consistent; #远程地址做个hash
server isrm-dev-public.obs.cn-east-2.myhuaweicloud.com:443;
}
server {
listen 80;
listen 443;
proxy_connect_timeout 1s; #后端链接空闲超时断开
proxy_timeout 10s; #后端连接超时时间
proxy_pass $upstream_name;
}
}

````
--------------------------------------------------------------------------------
##2.1 四层代理
传输层实现。
使用NAT技术，即网络地址转换，nginx相当于一个路由器。请求进来的时候，nginx修改数据包里面的目标和源IP和端口，然后把数据包发向目标服务器，服务器处理完成后，nginx再做一次修改，返回给请求的客户端。
以上过程TCP三次握手是客户端直接和后端连接的，只不过在后端机器上看到的都是与代理机的IP的established而已。

##2.2 七层代理
应用层实现。
代理机必须要先和客户端三次握手后，才能得到7层（HTT层）的具体内容；然后根据设置的负载均衡规则选择特定的webserver，通过三次握手建立TCP连接，再把数据包发给webserver；服务器处理完成后，webserver把需要的数据发送给七层负载均衡设备，负载均衡设备再把数据发送给客户端。
以上过程nginx分别和客户端、服务端建立了连接，效率上比四层要低，但是能实现更智能的调度。

https://blog.csdn.net/weixin_39743511/article/details/112949904

https://www.nginx.com/resources/glossary/layer-4-load-balancing/

https://www.nginx.com/resources/glossary/?_ga=2.123855114.1087834227.1619075482-778670044.1600149189

https://zhuanlan.zhihu.com/p/339580398