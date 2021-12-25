#Nginx安装可选配置

该构建使用该configure命令进行配置。它定义了系统的各个方面，包括允许nginx用于连接处理的方法。最后它创建一个Makefile。该configure命令支持以下参数：
````
--prefix = path - 定义一个将保留服务器文件的目录。 同样的目录也将用于由configure设置的所有相对路径（除了库源的路径）和nginx.conf配置文件中。 它默认设置为/ usr / local / nginx目录。
--sbin-path = path - 设置nginx可执行文件的名称。 该名称仅在安装期间使用。 默认情况下，该文件被命名为prefix / sbin / nginx。
--conf-path = path - 设置nginx.conf配置文件的名称。 如果需要，通过在命令行参数-c文件中指定nginx，nginx始终可以使用不同的配置文件启动。 默认情况下，该文件名为prefix / conf / nginx.conf。
--pid-path = path - 设置将存储主进程的进程ID的nginx.pid文件的名称。 安装完成后，可以使用pid指令始终在nginx.conf配置文件中更改文件名。 默认情况下，该文件名为prefix / logs / nginx.pid。
--error-log-path = path - 设置主要错误，警告和诊断文件的名称。 安装完成后，可以使用error_log指令始终在nginx.conf配置文件中更改文件名。 默认情况下，该文件名为prefix / logs / error.log。
--http-log-path = path - 设置HTTP服务器的主要请求日志文件的名称。 安装完成后，可以使用access_log指令始终在nginx.conf配置文件中更改文件名。 默认情况下，该文件被命名为prefix / logs / access.log。
--build=name  - 设置一个可选的nginx构建名称。
--user = name - 设置其凭据将被工作进程使用的非特权用户的名称。 安装完成后，可以使用user指令始终在nginx.conf配置文件中更改该名称。 默认的用户名是nobody。
--group = name - 设置工作进程将使用其凭据的组的名称。 安装完成后，可以使用user指令始终在nginx.conf配置文件中更改该名称。 默认情况下，组名称设置为非特权用户的名称。
--with-select_module
--without-select_module- 启用或禁用构建允许服务器使用该select()方法的模块。如果平台似乎不支持更合适的方法，例如kqueue，epoll或/ dev / poll，则会自动构建此模块。

--with-poll_module --without-poll_module- 启用或禁用构建允许服务器使用该poll()方法的模块。如果平台似乎不支持更合适的方法，例如kqueue，epoll或/ dev / poll，则会自动构建此模块。
--without-http_gzip_module - 禁用构建压缩HTTP服务器响应的模块。需要zlib库来构建和运行此模块。
--without-http_rewrite_module - 禁止构建允许HTTP服务器重定向请求并更改请求URI的模块。PCRE库需要构建和运行该模块。
--without-http_proxy_module  - 禁用构建HTTP服务器代理模块。
--with-http_ssl_module - 可以构建一个模块，将HTTPS协议支持添加到HTTP服务器。该模块不是默认生成的。OpenSSL库是构建和运行该模块所必需的。
--with-pcre=path - 设置PCRE库源的路径。图书馆发行版（版本4.4  -  8.41）需要从PCRE网站下载并提取。其余的由nginx的./configure和make。该库是位置指令和ngx_http_rewrite_module模块支持正则表达式所必需的。
--with-pcre-jit  - 用“即时编译”支持（1.1.12，pcre_jit指令）构建PCRE库。
--with-zlib=path - 设置zlib库源的路径。库分发（版本1.1.3  -  1.2.11）需要从zlib站点下载并解压缩。其余的由nginx的./configure和make。该库是ngx_http_gzip_module模块所必需的。
--with-cc-opt=parameters - 设置将被添加到CFLAGS变量的附加参数。在FreeBSD下使用系统PCRE库时，--with-cc-opt="-I /usr/local/include"应该指定。如果select()需要增加支持的文件数量，也可以在这里指定如下：--with-cc-opt="-D FD_SETSIZE=2048"。
--with-ld-opt=parameters - 设置将在链接期间使用的其他参数。在FreeBSD下使用系统PCRE库时，--with-ld-opt="-L /usr/local/lib"应该指定。
````

````
参数使用示例（所有这些都需要输入一行）：
./configure
--sbin-path=/usr/local/nginx/nginx
--conf-path=/usr/local/nginx/nginx.conf
--pid-path=/usr/local/nginx/nginx.pid
--with-http_ssl_module
--with-pcre=../pcre-8.41
--with-zlib=../zlib-1.2.11
````
配置完成后，nginx被编译并使用安装make。