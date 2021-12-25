###如何防止未定义服务器名称的处理请求
###混合基于名称和基于IP的虚拟服务器
###一个简单的PHP站点配置
##基于名称的虚拟服务器
nginx首先决定哪个服务器应该处理请求。让我们从一个简单的配置开始，所有三个虚拟服务器在端口*：80上进行监听：
````
server {
listen      80;
server_name example.org www.example.org;
...
}

server {
listen      80;
server_name example.net www.example.net;
...
}

server {
listen      80;
server_name example.com www.example.com;
...
}
````
在这个配置中，nginx只测试请求的头部字段“HOST”以确定请求应该被路由到哪个服务器。 

如果它的值与任何服务器名称不匹配，或者请求根本没有包含这个头域，那么nginx会将请求路由到这个端口的默认服务器。 在上面的配置中，默认服务器是第一个 - 这是nginx的标准默认行为。 

它也可以明确地设置哪个服务器应该是默认的，在listen指令中使用default_server参数：
````
server {
listen      80 default_server;
server_name example.net www.example.net;
...
}
````
该default_server参数自0.8.21版开始可用。在早期版本中应该使用参数default。

请注意，默认服务器是侦听端口的属性，而不是服务器名称的属性。稍后再详细介绍。

##如何防止未定义服务器名称的处理请求
如果不应该允许没有“Host”头字段的请求，那么可以定义一个只删除请求的服务器：
````
server {
listen      80;
server_name "";
return      444;
}
````
这里，服务器名称被设置为一个空字符串，它将匹配没有“Host”头字段的请求，并返回一个特殊的nginx的非标准代码444来关闭连接。

从版本0.8.48开始，这是服务器名称的默认设置，因此server_name ""可以省略。在早期版本中，机器的主机名被用作默认服务器名称。

##混合基于名称和基于IP的虚拟服务器
让我们看看更复杂的配置，其中一些虚拟服务器在不同的地址上进行侦听：
````
server {
listen      192.168.1.1:80;
server_name example.org www.example.org;
...
}

server {
listen      192.168.1.1:80;
server_name example.net www.example.net;
...
}

server {
listen      192.168.1.2:80;
server_name example.com www.example.com;
...
}
````
在此配置中，nginx首先根据服务器块的listen指令测试请求的IP地址和端口。 然后，它会根据匹配IP地址和端口的服务器块的server_name条目测试请求的“HOST”头字段。 如果未找到服务器名称，则该请求将由默认服务器处理。

例如，在192.168.1.1:80端口收到的www.example.com请求将由192.168.1.1:80端口的默认服务器处理，即由第一台服务器处理，因为没有www.example .com为此端口定义。

如前所述，默认服务器是侦听端口的属性，可以为不同的端口定义不同的默认服务器：
````
server {
listen      192.168.1.1:80;
server_name example.org www.example.org;
...
}

server {
listen      192.168.1.1:80 default_server;
server_name example.net www.example.net;
...
}

server {
listen      192.168.1.2:80 default_server;
server_name example.com www.example.com;
...
}
````
##一个简单的PHP站点配置
现在让我们看看nginx如何选择一个位置来处理一个典型的简单PHP站点的请求：
````
server {
listen      80;
server_name example.org www.example.org;
root        /data/www;

    location / {
        index   index.html index.php;
    }

    location ~* \.(gif|jpg|png)$ {
        expires 30d;
    }

    location ~ \.php$ {
        fastcgi_pass  localhost:9000;
        fastcgi_param SCRIPT_FILENAME
                      $document_root$fastcgi_script_name;
        include       fastcgi_params;
    }
}
````
nginx首先搜索由文字字符串给出的最具体的前缀位置，而不管列出的顺序如何。在上面的配置中，唯一的前缀位置是“ /”，并且因为它匹配任何请求，它将被用作最后的手段。然后nginx按照配置文件中列出的顺序检查正则表达式给出的位置。第一个匹配表达式会停止搜索，nginx将使用此位置。如果没有正则表达式匹配请求，那么nginx使用前面找到的最具体的前缀位置。

请注意，所有类型的位置仅测试没有参数的请求行的URI部分。这是因为查询字符串中的参数可能以多种方式给出，例如：

    /index.php?user=john&page=1
    /index.php?page=1&user=john
此外，任何人都可以在查询字符串中请求任何内容

    /index.php?page=1&something+else&user=john
现在我们来看看如何在上面的配置中处理请求：

请求“/logo.gif”首先由前缀位置“/”匹配，然后由正则表达式“\。（gif | jpg | png）$”匹配，因此它由后一个位置处理。 使用指令“root / data / www”将请求映射到文件/data/www/logo.gif，并将文件发送到客户端。

一个请求“/index.php”也被前缀位置“/”匹配，然后由正则表达式“\。（php）$”匹配。 因此，它由后一个位置处理，并将请求传递给侦听localhost：9000的FastCGI服务器。 fastcgi_param指令将FastCGI参数SCRIPT_FILENAME设置为“/data/www/index.php”，FastCGI服务器执行该文件。 变量$ document_root等于root指令的值，变量$ fastcgi_script_name等于请求URI，即“/index.php”。

请求“/about.html”仅由前缀位置“/”匹配，因此它在此位置处理。 使用指令“root / data / www”将请求映射到文件/data/www/about.html，并将文件发送到客户端。

处理请求“/”更复杂。 它仅由前缀位置“/”匹配，因此它由此位置处理。 然后，index指令根据其参数和“root / data / www”指令测试索引文件的存在性。 如果文件/data/www/index.html不存在，并且文件/data/www/index.php存在，那么该指令将内部重定向到“/index.php”，并且nginx再次搜索位置为 如果请求是由客户发送的。 正如我们之前所见，重定向的请求最终将由FastCGI服务器处理。