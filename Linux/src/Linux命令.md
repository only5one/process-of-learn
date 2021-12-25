#处理目录的常用命令
ls（英文全拼：list files）: 列出目录及文件名

cd（英文全拼：change directory）：切换目录

pwd（英文全拼：print work directory）：显示目前的目录

mkdir（英文全拼：make directory）：创建一个新的目录

rmdir（英文全拼：remove directory）：删除一个空的目录

cp（英文全拼：copy file）: 复制文件或目录

rm（英文全拼：remove）: 删除文件或目录

mv（英文全拼：move file）: 移动文件与目录，或修改文件与目录的名称

使用man [命令] 可以查看各个命令的使用方法

#Linux 文件内容查看
Linux系统中使用以下命令来查看文件的内容：

cat  由第一行开始显示文件内容

tac  从最后一行开始显示，可以看出 tac 是 cat 的倒着写！

nl   显示的时候，顺道输出行号！

more 一页一页的显示文件内容

less 与 more 类似，但是比 more 更好的是，他可以往前翻页！

head 只看头几行

tail 只看尾巴几行

使用man [命令] 可以查看各个命令的使用方法

#Linux 磁盘管理
Linux 磁盘管理好坏直接关系到整个系统的性能问题。

Linux 磁盘管理常用三个命令为 df、du 和 fdisk。

df（英文全称：disk full）：列出文件系统的整体磁盘使用量

du（英文全称：disk used）：检查磁盘空间使用量

fdisk：用于磁盘分区

使用man [命令] 可以查看各个命令的使用方法

#Linux yum 命令
yum（ Yellow dog Updater, Modified）是一个在 Fedora 和 RedHat 以及 SUSE 中的 Shell 前端软件包管理器。

基于 RPM 包管理，能够从指定的服务器自动下载 RPM 包并且安装，可以自动处理依赖性关系，并且一次安装所有依赖的软件包，无须繁琐地一次次下载、安装。

yum 提供了查找、安装、删除某一个、一组甚至全部软件包的命令，而且命令简洁而又好记。

yum 语法：
        yum [options] [command] [package ...]
        options：可选，选项包括-h（帮助），-y（当安装过程提示选择全部为 "yes"），-q（不显示安装的过程）等等。
        command：要进行的操作。
        package：安装的包名。

**yum常用命令**
1. 列出所有可更新的软件清单命令：yum check-update

2. 更新所有软件命令：yum update

3. 仅安装指定的软件命令：yum install <package_name>

4. 仅更新指定的软件命令：yum update <package_name>

5. 列出所有可安裝的软件清单命令：yum list

6. 删除软件包命令：yum remove <package_name>

7. 查找软件包命令：yum search <keyword>

8. 清除缓存命令:
yum clean packages: 清除缓存目录下的软件包
   
yum clean headers: 清除缓存目录下的 headers

yum clean oldheaders: 清除缓存目录下旧的 headers

yum clean, yum clean all (= yum clean packages; yum clean oldheaders) :清除缓存目录下的软件包及旧的 headers