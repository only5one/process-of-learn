    perl(Data::Dumper) 被 mysql-community-test-5.7.29-1.el7.x86_64 需要
解决：yum -y install autoconf	

    mysql-community-server(x86-64) >= 5.7.9 被 mysql-community-test-5.7.29-1.el7.x86_64 需要
解决：调换server和test安装顺序

启动mysql后快速报错
原因：复制配置文件时，直接复制会使配置文件的第一句话失去#For a 这5个字符，没有#导致注释被读取

启动mysql后过一会报错
原因：磁盘空间太小，需要扩容