###资源需求：
两服务器，防火墙互通
一个VIP
部署内容：两个MySQL、一组keepalived

操作前请检查服务器CPU、内存、磁盘挂载、网络环境等信息。

##基础服务安装（两台服务器都需要操作）
清理及下载依赖
```rpm -qa|grep "mariadb-libs"|xargs -i rpm -e --nodeps {}  或者：
yum -y remove mariadb-*
rpm -qa|grep mysql|xargs -i rpm -e --nodeps {}
yum install libaio perl net-tools perl-JSON.noarch make gcc-c++ cmake bison-devel ncurses-devel numactl -y
```

##下载安装包安装
```wget -c https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.29-1.el7.x86_64.rpm-bundle.tar
tar -xvf mysql-5.7.29-1.el7.x86_64.rpm-bundle.tar
rpm -ivh mysql-community-common-5.7.29-1.el7.x86_64.rpm
rpm -ivh mysql-community-libs-5.7.29-1.el7.x86_64.rpm
rpm -ivh mysql-community-libs-compat-5.7.29-1.el7.x86_64.rpm
rpm -ivh mysql-community-embedded-5.7.29-1.el7.x86_64.rpm
rpm -ivh mysql-community-embedded-compat-5.7.29-1.el7.x86_64.rpm
rpm -ivh mysql-community-devel-5.7.29-1.el7.x86_64.rpm
rpm -ivh mysql-community-embedded-devel-5.7.29-1.el7.x86_64.rpm
rpm -ivh mysql-community-client-5.7.29-1.el7.x86_64.rpm
rpm -ivh mysql-community-server-5.7.29-1.el7.x86_64.rpm
rpm -ivh mysql-community-test-5.7.29-1.el7.x86_64.rpm
```
##设置mysql开机自启动
```
systemctl enable mysqld.service
```

##修改配置文件
```
[mysqld]
########basic settings########
# 每个节点的server-id一定要设置不同
server-id = 135
port = 3306
user = mysql
bind_address = 0.0.0.0    
skip_name_resolve = 1
# 指定数据文件路径
datadir = /data/mysql
log_error = error.log
character-set-server=utf8
lower_case_table_names=1
max_connections=1000
max_connect_errors=10000
innodb_buffer_pool_size=10G
back_log=900
open_files_limit=102400
thread_cache_size=128
table_open_cache=1024
innodb_buffer_pool_instances=4
innodb_flush_method=O_DIRECT
innodb_log_file_size=1073741824
#######replication settings########
master_info_repository = TABLE
relay_log_info_repository = TABLE
# MySQL复制是基于binlog日志的
log_bin = master-bin
sync_binlog = 1
log_slave_updates
binlog_format = row
relay_log = relay.log
relay_log_recovery = 1
slave_skip_errors = ddl_exist_errors
binlog_format = mixed
auto-increment-increment = 2    
auto-increment-offset = 1
######semi sync replication settings########
# 设置插件目录路径
plugin_dir=/usr/lib64/mysql/plugin
# 加载插件
plugin_load = "rpl_semi_sync_master=semisync_master.so;rpl_semi_sync_slave=semisync_slave.so"
# 开启master semi sync replication
rpl_semi_sync_master_enabled = 1
# 开启slave semi sync replication
rpl_semi_sync_slave_enabled = 1
# 等待5秒无ack应答自动切换为异步模式
rpl_semi_sync_master_timeout = 5000
# 开启lossless replication
rpl_semi_sync_master_wait_point= AFTER_SYNC
# 至少有1个slave接收到日志
rpl_semi_sync_master_wait_for_slave_count = 1
```
##启动MySQL

    systemctl start mysqld

##获得root初始登陆密码
```
cat /var/log/mysqld.log | grep password
cat /data/mysql/error.log | grep password
```
##进入数据库
    mysql -uroot -p'上一步获取的初始密码'

##修改密码
    Set password for 'root'@'localhost'=password('HandRoot#2020');

##修改root可以远程ip登录
```
update mysql.user set host = '%' where user = 'root';
flush privileges;
```

##设置事务模式：（根据需要设置）
###查看事务模式：
```
SELECT @@tx_isolation;
//设置read uncommitted级别：
set global transaction isolation level read uncommitted;
//设置read committed级别：
set global transaction isolation level read committed;
//设置repeatable read级别：
set global transaction isolation level repeatable read;
//设置serializable级别：
set global transaction isolation level serializable;
```
#配置双主模式

##两台服务器创建Slave账号，并授权
```
Create user 'slave'@'%' identified by 'Slavel*FFghg6';
GRANT REPLICATION SLAVE ON *.* TO 'slave'@'%'  identified by 'Slavel*FFghg6';
```
##登录两节点数据库查看master状态并记录下File和Position参数值
    show master status;
两节点分别执行以下命令（注意修改参数）
10.185.101.177执行：
```
change master to master_host='10.185.101.178', master_port=3306, master_user='slave', master_password='Slavel*FFghg6', master_log_file='master-bin.000002', master_log_pos=1448;
start slave;
show slave status\G
```
--------
```
10.185.101.178执行：
change master to master_host='10.185.101.177', master_port=3306, master_user='slave', master_password='Slavel*FFghg6', master_log_file='master-bin.000002', master_log_pos=1605;
start slave;
show slave status\G
````
![img.png](../../day06/images/img.png)
当两个数据的这两参数均为YES时说明双主配置成功。
