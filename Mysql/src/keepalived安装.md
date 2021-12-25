#配置keepalived
##安装keepalived
````
yum install -y libnfnetlink-devel kernel-devel* openssl-* popt-devel lrzsz openssh-clients libnl libnl-devel popt gcc-c++
wget -c https://www.keepalived.org/software/keepalived-1.2.15.tar.gz
./configure --prefix=/data/keepalived #--prefix=安装地址
make && make install
或者
yum -y install keepalived
````
##复制/sbin/keepalived到/usr/sbin下
    cp /data/keepalived/sbin/keepalived /usr/sbin/
##keepalived默认会读取/etc/keepalived/keepalived.conf配置文件
```
mkdir /etc/keepalived
cp /data/keepalived/etc/keepalived/keepalived.conf /etc/keepalived/keepalived.conf
```
##复制sysconfig文件到/etc/sysconfig下
    cp /data/keepalived/etc/sysconfig/keepalived /etc/sysconfig/
##复制启动脚本到/etc/init.d下
    cd /data/keepalived
    cp ./keepalived/etc/rc.d/keepalived /etc/init.d/
    chmod 755 /etc/init.d/keepalived
##keepalived配置文件（/etc/keepalived/keepalived.conf）：
192.168.13.3节点：
```
! Configuration File for keepalived

global_defs {
   router_id prod-db1
}
vrrp_instance VI_1 {
    state BACKUP # 两个节点都为BACKUP状态，根据优先级大小判断谁为MASTER
    interface ens33	#改成实际网卡名
    virtual_router_id 51
    priority 100
    advert_int 1
    nopreempt # 非抢占模式
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    # 虚拟IP池
    virtual_ipaddress {
       192.168.13.180
    }
   unicast_src_ip  192.168.13.3  #单播的源地址，写本机上的ip即可
    unicast_peer {
        192.168.13.4   #如果有多个主机组成集群，把其它主机ip都写上
    }
}
virtual_server 192.168.13.180 3306 {
     delay_loop 2
     lb_algo wrr
     lb_kind DR
     persistence_timeout 60
     protocol TCP
     real_server 192.168.13.3 3306 {
         weight 3
         notify_down /etc/keepalived/mysql.sh # 当mysql服务down了之后，执行的脚本
         TCP_CHECK {
             connect_timeout 10   # mysql连接超时时长（秒）
             nb_get_retry 3       # mysql服务连接失败，重试次数
             delay_before_retry 3 #每隔3秒检测一次mysql服务是否可用
             connect_port 3306
         }
     }
}
```

192.168.13.4节点：
```
! Configuration File for keepalived

global_defs {
   router_id prod-db1
}
vrrp_instance VI_1 {
    state BACKUP # 两个节点都为BACKUP状态，根据优先级大小判断谁为MASTER
    interface ens33	#改成实际网卡名
    virtual_router_id 51
    priority 100
    advert_int 1
    nopreempt # 非抢占模式
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    # 虚拟IP池
    virtual_ipaddress {
       192.168.13.180
    }
   unicast_src_ip  192.168.13.4  #单播的源地址，写本机上的ip即可
    unicast_peer {
        192.168.13.3   #如果有多个主机组成集群，把其它主机ip都写上
    }
}
virtual_server 192.168.13.180 3306 {
     delay_loop 2
     lb_algo wrr
     lb_kind DR
     persistence_timeout 60
     protocol TCP
     real_server 192.168.13.4 3306 {
         weight 3
         notify_down /etc/keepalived/mysql.sh # 当mysql服务down了之后，执行的脚本
         TCP_CHECK {
             connect_timeout 10   # mysql连接超时时长（秒）
             nb_get_retry 3       # mysql服务连接失败，重试次数
             delay_before_retry 3 #每隔3秒检测一次mysql服务是否可用
             connect_port 3306
         }
     }
}
```
##两个服务器分别添加脚本/etc/keepalived/mysql.sh：
内容为：
```
#!/bin/bash
systemctl restart keepalived
```
（注意：编译安装需要自己将keepalived配置成系统服务，才能使用如上命令重启）
给脚本授予执行权限:
chmod +x /etc/keepalived/mysql.sh

##Keepalived测试VIP漂移：
###模拟Keepalived宕机：
将有VIP的节点的keepalived关闭，查看VIP是否漂移到另一节点。

防止多余的VIP漂移，Keepalived采取不争抢模式，所以重启刚刚关闭的Keepalived的服务器并不会获得VIP。

然后进行反向测试。

具体：使用ip addr list查看vip在哪台虚拟机，停止服务，换到另外一台虚拟机上查看，vip是否已经更换。

mysql也是同理。
###模拟MySql宕机：
将有VIP的节点的MySql关闭，查看VIP是否漂移到另一节点。

防止多余的VIP漂移，Keepalived采取不争抢模式，所以重启刚刚关闭的MySql的服务器并不会获得VIP。

然后在另一节点进行相同操作测试。

测试完毕记得检查Keepalived和Mysql的状态是否都是running，如不是请启动服务。
```
systemctl status mysqld
systemctl status keepalived
systemctl start mysqld
systemctl start keepalived
```
--------------------------------------------------------------------------------
编译安装keepalived报错：
check/check_ssl.o：在函数‘build_ssl_ctx’中：
/data/software/keepalived-1.2.15/keepalived/check/check_ssl.c:68：对‘OPENSSL_init_ssl’未定义的引用
check/check_ssl.o：在函数‘init_ssl_ctx’中：
check_ssl.c:(.text+0x2a1)：对‘OPENSSL_init_ssl’未定义的引用
check_ssl.c:(.text+0x2ce)：对‘TLS_method’未定义的引用
collect2: 错误：ld 返回 1
make[1]: *** [all] 错误 1
make[1]: 离开目录“/data/software/keepalived-1.2.15/keepalived”
make: *** [all] 错误 2
解决：换yum安装