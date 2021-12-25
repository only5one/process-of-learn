###nginx支持以下命令行参数：
````
-?| -h - 打印命令行参数的帮助。
-c file- 使用替代配置file而不是默认文件。
-g directives- 设置全局配置指令，例如，nginx -g“pid /var/run/nginx.pid; worker_processes sysctl -n hw.ncpu;”
-p prefix- 设置nginx路径前缀，即保留服务器文件的目录（默认值是/usr/local/nginx）。
-q  - 在配置测试期间抑制非错误消息。
-s signal- 向主进程发送一个信号。参数信号可以是以下之一：
stop  - 快速关闭
quit  - 较慢地关闭
reload  - 重新加载配置，使用新配置启动新的工作进程，正常关闭旧的工作进程。
reopen  - 重新打开日志文件
-t  - 测试配置文件：nginx检查配置的正确语法，然后尝试打开配置中引用的文件。
-T- 与-t相同，但另外将配置文件转储到标准输出（1.9.2）。
-v  - 打印nginx版本。
-V  - 打印nginx版本，编译器版本和配置参数。
```` 