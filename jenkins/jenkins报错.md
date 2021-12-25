##安装jenkins后构建工程，执行报mvn command not found
在使用Jenkins进行构建项目时，绝大部分会使用到maven、nodejs相关的命令， 服务器已经安装好了maven、nodejs相关程序，并且在jenkins配置了maven但是在Jenkins shell、pipeline script中使用mvn、npm命令还是报command not found的错误

原因:jenkins里面没有我们服务器path的环境变量，所以才会出现找不到命令的错误。

解决方案:
````
在Jenkins部署服务器输出一下PATH路径： echo $PATH
在Jenkins -> Manager Jenkins -> Configure System中添加环境变量:
找到 Global properties，勾选中Environment variables，一个PATH变量以后保存
参考:https://blog.csdn.net/qq_32352777/article/details/109267692
````

