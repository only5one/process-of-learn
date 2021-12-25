#配置expires
expires起到控制页面缓存的作用，合理的配置expires可以减少很多服务器的请求

要配置expires，可以在http段中或者server段中或者location段中加入
````
location ~ \.(gif|jpg|jpeg|png|bmp|ico)$ {
root /var/www/img/;
expires 30d;
}
````
控制图片等过期时间为30天，当然这个时间可以设置的更长。具体视情况而定
````
location ~ \.(wma|wmv|asf|mp3|mmf|zip|rar|swf|flv)$ {
root /var/www/upload/;
expires max;
}
````

expires 指令可以控制 HTTP 应答中的“ Expires ”和“ Cache-Control ”的头标（起到控制页面缓存的作用）

####语法：expires [time|epoch|max|pff]

####默认值：off

expires指令控制HTTP应答中的“Expires”和“Cache-Control”Header头部信息，启动控制页面缓存的作用
time:可以使用正数或负数。“Expires”头标的值将通过当前系统时间加上设定time值来设定。

####time值还控制"Cache-Control"的值：

####负数表示no-cache

####正数或零表示max-age=time

####epoch：指定“Expires”的值为 1 January,1970,00:00:01 GMT

####max:指定“Expires”的值为31 December2037 23:59:59GMT,"Cache-Control"的值为10年。

####-1：指定“Expires”的值为当前服务器时间-1s，即永远过期。

####off：不修改“Expires”和"Cache-Control"的值

expires使用了特定的时间，并且要求服务器和客户端的是中严格同步。

而Cache-Control是用max-age指令指定组件被缓存多久。

对于不支持http1.1的浏览器，还是需要expires来控制。所以最好能指定两个响应头。但HTTP规范规定max-age指令将重写expires头。