#简单代理

###启动，停止和重新加载配置
###配置文件的结构
###提供静态内容
###设置简单的代理服务器
###设置FastCGI代理

nginx有一个主进程和几个工作进程。主进程的主要目的是读取和评估配置，并维护工作进程。工作进程会处理请求的实际处理。nginx使用基于事件的模型和依赖于操作系统的机制来高效地在工作进程间分配请求。工作进程的数量在配置文件中定义，并且可以针对给定配置进行修复或自动调整为可用CPU核心的数量（请参阅worker_processes）。

nginx及其模块的工作方式在配置文件中确定。默认情况下，该配置文件被命名nginx.conf，并放入目录/usr/local/nginx/conf，/etc/nginx或/usr/local/etc/nginx。

启动，停止和重新加载配置
要启动nginx，请运行可执行文件。一旦nginx启动，就可以通过调用带有-s参数的可执行文件来控制它。使用以下语法：

nginx -s signal
当信号 可以是下列之一：

stop  - 快速关机
quit  - 较慢地关机
reload  - 重新加载配置文件
reopen  - 重新打开日志文件
例如，要停止nginx进程并等待工作进程完成当前请求的服务，可以执行以下命令：

    nginx -s quit
这个命令应该在启动nginx的同一个用户下执行。

在重新加载配置的命令发送到nginx或重新启动之前，配置文件中所做的更改将不会应用。要重新加载配置，请执行：

    nginx -s reload
主进程收到重新加载配置的信号后，会检查新配置文件的语法有效性，并尝试应用其中提供的配置。如果这是成功的，则主进程启动新的工作进程并将消息发送给旧工作进程，请求它们关闭。否则，主进程将回滚更改并继续使用旧配置。旧工作进程接收关闭的命令，停止接受新的连接并继续服务当前的请求，直到所有这样的请求得到服务。之后，老员工流程退出。

一个信号也可以在Unix工具（例如kill实用程序）的帮助下发送到nginx进程。在这种情况下，信号直接发送给具有给定进程ID的进程。默认情况下，nginx主进程的进程ID被写入nginx.pid目录/usr/local/nginx/logs或/var/run。例如，如果主进程ID是1628，要发送导致nginx正常关闭的QUIT信号，请执行：

    kill -s QUIT 1628
要获取所有正在运行的nginx进程的列表，ps可以按照以下方式使用该实用程序：

    ps -ax | grep nginx

###配置文件的结构
nginx包含由配置文件中指定的指令控制的模块。指令分为简单指令和块指令。一个简单的指令由名称和参数组成，由空格分隔并以分号（;）结束。块指令与简单指令具有相同的结构，但不是以分号结尾，而是以一系列由大括号（{和}）包围的附加指令结束。如果block指令可以在大括号内包含其他指令，则它被称为上下文（例如：事件，http，服务器和位置）。

放置在任何上下文之外的配置文件中的指令被认为是在主要上下文中。在events和http指令驻留在main上下文server中http，并location在server。

\#标志后面的其余部分被认为是备注。

###提供静态内容
一个重要的Web服务器任务是提供文件（如图像或静态HTML页面）。您将实施一个示例，根据请求，文件将从不同的本地目录/data/www（可能包含HTML文件）和/data/images（包含图像）提供。这将需要编辑配置文件并在两个位置块的http块中设置一个服务器块。

首先，创建/data/www目录，并将index.html包含任何文本内容的文件放入其中，并创建/data/images目录并放置一些图像。

接下来，打开配置文件。默认的配置文件已经包含了几个server块的例子，大部分都是注释掉的。现在注释掉所有这些块并开始一个新server块：
````
http {
server {
}
}
````
通常，配置文件可能包括几个服务器模块，这些服务器模块可以通过它们侦听的端口和服务器名称来区分。 一旦nginx决定哪个服务器处理请求，它会根据服务器块内定义的位置指令的参数来测试请求标头中指定的URI。

将以下location块添加到server块中：
````
location / {
root /data/www;
}
````
该location块指定/与来自请求的URI相比较的前缀“ ”。对于匹配请求，URI将被添加到根指令中指定的路径，即为了/data/www在本地文件系统上形成请求文件的路径。如果有几个匹配location块，nginx会选择最长前缀的块。location上面的块提供了最短的前缀，长度为1，所以只有当所有其他location块未能提供匹配时，才会使用此块。

接下来，添加第二个location块：
````
location /images/ {
root /data;
}
````
它将匹配以开头的请求/images/（location /也匹配这样的请求，但具有较短的前缀）。

server块的结果配置应该如下所示：
````
server {
    location / {
    root /data/www;
}

    location /images/ {
        root /data;
    }
}
````
这已经是一个服务器的工作配置，它监听标准端口80，并且可以在本地机器上访问http://localhost/。为了响应以URI开头的请求/images/，服务器将从/data/images目录发送文件。例如，响应http://localhost/images/example.png请求，nginx将发送/data/images/example.png文件。如果这样的文件不存在，nginx将发送一个指示404错误的响应。具有不是以URI开头的请求/images/将被映射到该/data/www目录。例如，响应http://localhost/some/example.html请求，nginx将发送/data/www/some/example.html文件。

要应用新的配置，请启动nginx，如果它尚未启动或发送reload信号到nginx的主进程，执行：

    nginx -s reload
在一些情况下不按预期工作，您可以尝试找出原因access.log和error.log目录中的文件/usr/local/nginx/logs或/var/log/nginx。

###设置简单的代理服务器
nginx的一个常用用途是将其设置为代理服务器，这意味着服务器接收请求，将它们传递给代理服务器，从中检索响应并将它们发送给客户端。

我们将配置一个基本的代理服务器，该服务器为来自本地目录的文件的图像请求提供服务，并将所有其他请求发送给代理服务器。在这个例子中，两个服务器将在单个nginx实例上定义。

首先，通过向servernginx的配置文件添加一个更多的块并使用以下内容定义代理服务器：
````
    server {
    listen 8080;
    root /data/up1;

    location / {
    }
}
````
这将是一个监听端口8080的简单服务器（以前，自从使用标准端口80以来，listen指令未被指定）并将所有请求映射到本地文件系统上的/ data / up1目录。 创建该目录并将index.html文件放入其中。 请注意，root指令放置在服务器上下文中。 当选择用于请求的位置块不包括自己的根指令时，使用这种根指令。

接下来，使用上一节中的服务器配置并对其进行修改，使其成为代理服务器配置。 在第一个位置块中，将proxy_pass指令与参数中指定的代理服务器的协议，名称和端口（在本例中为http：// localhost：8080）相加：
````
server {
location / {
proxy_pass http://localhost:8080;
}

    location /images/ {
        root /data;
    }
}
````
我们将修改第二个location块，它将当前带有/images/前缀的请求映射到目录下的/data/images文件，以使其与具有典型文件扩展名的图像的请求匹配。修改的location块如下所示：
````
location ~ \.(gif|jpg|png)$ {
root /data/images;
}
````
该参数是一个匹配以.gif，.jpg或.png结尾的所有URI的正则表达式。 正则表达式应该以〜开头。 相应的请求将被映射到/ data / images目录。

当nginx选择一个位置块来为请求提供服务时，它首先检查指定前缀的位置指令，记住具有最长前缀的位置，然后检查正则表达式。 如果与正则表达式匹配，nginx将选择此位置，否则，它会选择之前记住的位置。

代理服务器的结果配置如下所示：
````
server {
location / {
proxy_pass http://localhost:8080/;
}

    location ~ \.(gif|jpg|png)$ {
        root /data/images;
    }
}
````
该服务器将过滤以.gif，.jpg或.png结尾的请求，并将它们映射到/ data / images目录（通过将URI添加到root指令的参数）并将所有其他请求传递给上面配置的代理服务器。

要应用新配置，请reload按照前面部分所述将信号发送到nginx。

还有许多指令可用于进一步配置代理连接。

###设置FastCGI代理
nginx可用于将请求路由到运行由各种框架和编程语言（如PHP）构建的应用程序的FastCGI服务器。

与FastCGI服务器一起工作的最基本的nginx配置包括使用fastcgi_pass指令而不是proxy_pass指令，以及fastcgi_param指令来设置传递给FastCGI服务器的参数。 假设FastCGI服务器可以通过localhost：9000访问。 以上一节中的代理配置为基础，将proxy_pass指令替换为fastcgi_pass指令并将参数更改为localhost：9000。 在PHP中，SCRIPT_FILENAME参数用于确定脚本名称，QUERY_STRING参数用于传递请求参数。 最终的配置将是：
````
server {
location / {
fastcgi_pass  localhost:9000;
fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
fastcgi_param QUERY_STRING    $query_string;
}

    location ~ \.(gif|jpg|png)$ {
        root /data/images;
    }
}
````
这将建立一个服务器，将除了静态图像请求之外的所有请求路由到localhost:9000通过FastCGI协议运行的代理服务器。