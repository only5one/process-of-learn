#开启网络
**修改/etc/sysconfig/network-scripts/ifcfg-ensXXXX文件**

设置  BOOTPROTO=dhcp，ONBOOT=yes

重启network

#1.设置IP地址、子网掩码和网关
**修改/etc/sysconfig/network-scripts/ifcfg-ens***

    BOOTPROTO=static

    IPADDR=192.168.xx.x

    NETMASK=255.255.255.0

    GATEWAY=192.168.XX.2

    ONBOOT=yes

xx更改为与主机虚拟适配器一致

#2.设置DNS
修改/etc/resolv.conf
nameserver [网关]

#3.设置主机名
**修改/etc/sysconfig/network**

NETWORKING=yes

HOSTNAME=xxxxxx

**修改/etc/hostname**

xxxxxx

#4.设置主机虚拟适配器
设置主机虚拟适配器ipv4协议ip地址与虚拟机IP地址在同一范围内，且不与网关和广播地址相同