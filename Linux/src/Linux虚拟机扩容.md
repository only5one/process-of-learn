首先使用VMware调整虚拟机最大硬盘容量
###打开虚拟机
使用
        
    df -h
查看虚拟机硬盘挂载情况

###使用Linux自带的磁盘管理工具
    fdisk /dev/sda
m可以查看帮助
p查看磁盘
n创建分区，然后使用p初始化新分区，默认为3，起始扇区和终点扇区均使用默认
p查看磁盘
w从内存将设置写入硬盘保存

###重启
    reboot
    pvcreate /dev/sda3
    vgextend /dev/mapper/centos /dev/sda3
    lvextend -L +xxG /dev/mapper/centos-root /dev/sda3
xxG填入调整的大小，如果扇区不够，可以改为(xx-1).

    resize2fs /dev/mapper/centos-root
如果失败则

    cat /etc/fstab | grep centos-root
查看磁盘是否为xfs
如果是则    

    xfs_growfs /dev/mapper/centos-root
完成后 

    dh -h
查看磁盘挂载情况