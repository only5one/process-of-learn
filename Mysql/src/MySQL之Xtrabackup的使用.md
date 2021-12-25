#一、Xtrabackup介绍
XtraBackup(PXB) 工具是 Percona 公司用 perl 语言开发的一个用于 MySQL 数据库物理热备的备份工具，支持 MySQl（Oracle）、Percona Server 和 MariaDB，并且全部开源。

##1.1 Xtrabackup 优点


1）备份速度快，物理备份可靠

2）备份过程不会打断正在执行的事务（无需锁表）

3）能够基于压缩等功能节约磁盘空间和流量

4）自动备份校验

5）还原速度快

6）可以流传将备份传输到另外一台机器上

7）在不增加服务器负载的情况备份数据


##1.2 Xtrabackup备份原理

备份开始时首先会开启一个后台检测进程，实时检测mysq redo的变化，一旦发现有新的日志写入，立刻将日志记入后台日志文件xtrabackup_log中，之后复制innodb的数据文件一系统表空间文件ibdatax，复制结束后，将执行flush tables with readlock,然后复制.frm MYI MYD等文件，最后执行unlock tables,最终停止xtrabackup_log。

##1.3 增量备份介绍:

1)、首先完成一个完全备份，并记录下此时检查点LSN；

2)、然后增量备份时，比较表空间中每个页的LSN是否大于上次备份的LSN，若是则备份该页并记录当前检查点的LSN。

**增量备份优点：**

1)、数据库太大没有足够的空间全量备份，增量备份能有效节省空间，并且效率高；

2)、支持热备份，备份过程不锁表（针对InnoDB而言），不阻塞数据库的读写；

3)、每日备份只产生少量数据，也可采用远程备份，节省本地空间；

4)、备份恢复基于文件操作，降低直接对数据库操作风险；

5)、备份效率更高，恢复效率更高。

#二、 Xtrabackup 安装
yum 源中的 XtraBackup 版本较低不能备份高版本的 MySQL 数据库，所以不建议使用 yum 的方式安装 XtraBackup。

根据自己的 MySQL 版本前往官网 下载对应的安装包,我的MySQL 版本是5.7的这里我用的是2.4.9的版本。

##2.1 下载 rpm 文件
    wget https://www.percona.com/downloads/XtraBackup/Percona-XtraBackup-2.4.9/binary/redhat/7/x86_64/percona-xtrabackup-24-2.4.9-1.el7.x86_64.rpm
    rpm -ivh percona-xtrabackup-24-2.4.9-1.el7.x86_64.rpm
##2.2 安装
先安装依赖：
````
wget http://mirror.centos.org/centos/7/extras/x86_64/Packages/libev-4.15-7.el7.x86_64.rpm
rpm -ivh libev-4.15-7.el7.x86_64.rpm
yum install perl-DBI
yum -y install perl perl-devel libaio libaio-devel perl-Time-HiRes perl-DBD-MySQL
yum -y install perl-Digest-MD5
````
在安装xtrabackup:

    rpm -ivh percona-xtrabackup-24-2.4.9-1.el7.x86_64.rpm
查看安装版本：
xtrabackup -version
查看安装的路径：
rpm -ql percona-xtrabackup-24
/usr/bin/innobackupex								#xtrabackup 的软连接
/usr/bin/xbcloud
/usr/bin/xbcloud_osenv
/usr/bin/xbcrypt									#加密解密用
/usr/bin/xbstream									#类似于tar，是 Percona 自己实现的一种支持并发写的流文件格式
/usr/bin/xtrabackup
/usr/share/doc/percona-xtrabackup-24-2.4.9
/usr/share/doc/percona-xtrabackup-24-2.4.9/COPYING
/usr/share/man/man1/innobackupex.1.gz
/usr/share/man/man1/xbcrypt.1.gz
/usr/share/man/man1/xbstream.1.gz
/usr/share/man/man1/xtrabackup.1.gz

#三、Xtrabackup 命令说明
![img2.png](../../day06/images/img2.png)
参数说明:
```
--apply-log-only：prepare备份的时候只执行redo阶段，用于增量备份。
--backup：创建备份并且放入--target-dir目录中
--close-files：不保持文件打开状态，xtrabackup打开表空间的时候通常不会关闭文件句柄，目的是为了正确处理DDL操作。如果表空间数量非常巨大并且不适合任何限制，一旦文件不在被访问的时候这个选项可以关闭文件句柄.打开这个选项会产生不一致的备份。
--compact：创建一份没有辅助索引的紧凑备份
--compress：压缩所有输出数据，包括事务日志文件和元数据文件，通过指定的压缩算法，目前唯一支持的算法是quicklz.结果文件是qpress归档格式，每个xtrabackup创建的*.qp文件都可以通过qpress程序提取或者解压缩
--compress-chunk-size=#：压缩线程工作buffer的字节大小，默认是64K
--compress-threads=#：xtrabackup进行并行数据压缩时的worker线程的数量，该选项默认值是1，并行压缩（'compress-threads'）可以和并行文件拷贝('parallel')一起使用。例如:'--parallel=4 --compress --compress-threads=2'会创建4个IO线程读取数据并通过管道传送给2个压缩线程。
--create-ib-logfile：这个选项目前还没有实现，目前创建Innodb事务日志，你还是需要prepare两次。
--datadir=DIRECTORY：backup的源目录，mysql实例的数据目录。从my.cnf中读取，或者命令行指定。
--defaults-extra-file=[MY.CNF]：在global files文件之后读取，必须在命令行的第一选项位置指定。
--defaults-file=[MY.CNF]：唯一从给定文件读取默认选项，必须是个真实文件，必须在命令行第一个选项位置指定。
--defaults-group=GROUP-NAME：从配置文件读取的组，innobakcupex多个实例部署时使用。
--export：为导出的表创建必要的文件
--extra-lsndir=DIRECTORY：(for --bakcup):在指定目录创建一份xtrabakcup_checkpoints文件的额外的备份。
--incremental-basedir=DIRECTORY：创建一份增量备份时，这个目录是增量别分的一份包含了full bakcup的Base数据集。
--incremental-dir=DIRECTORY：prepare增量备份的时候，增量备份在DIRECTORY结合full backup创建出一份新的full backup。
--incremental-force-scan：创建一份增量备份时，强制扫描所有增在备份中的数据页即使完全改变的page bitmap数据可用。
--incremetal-lsn=LSN：创建增量备份的时候指定lsn。
--innodb-log-arch-dir：指定包含归档日志的目录。只能和xtrabackup --prepare选项一起使用。
--innodb-miscellaneous：从My.cnf文件读取的一组Innodb选项。以便xtrabackup以同样的配置启动内置的Innodb。通常不需要显示指定。
--log-copy-interval=#：这个选项指定了log拷贝线程check的时间间隔（默认1秒）。
--log-stream：xtrabakcup不拷贝数据文件，将事务日志内容重定向到标准输出直到--suspend-at-end文件被删除。这个选项自动开启--suspend-at-end。
--no-defaults：不从任何选项文件中读取任何默认选项,必须在命令行第一个选项。
--databases=#：指定了需要备份的数据库和表。
--database-file=#：指定包含数据库和表的文件格式为databasename1.tablename1为一个元素，一个元素一行。
--parallel=#：指定备份时拷贝多个数据文件并发的进程数，默认值为1。
--prepare：xtrabackup在一份通过--backup生成的备份执行还原操作，以便准备使用。
--print-default：打印程序参数列表并退出，必须放在命令行首位。
--print-param：使xtrabackup打印参数用来将数据文件拷贝到datadir并还原它们。
--rebuild_indexes：在apply事务日志之后重建innodb辅助索引，只有和--prepare一起才生效。
--rebuild_threads=#：在紧凑备份重建辅助索引的线程数，只有和--prepare和rebuild-index一起才生效。
--stats：xtrabakcup扫描指定数据文件并打印出索引统计。
--stream=name：将所有备份文件以指定格式流向标准输出，目前支持的格式有xbstream和tar。
--suspend-at-end：使xtrabackup在--target-dir目录中生成xtrabakcup_suspended文件。在拷贝数据文件之后xtrabackup不是退出而是继续拷贝日志文件并且等待知道xtrabakcup_suspended文件被删除。这项可以使xtrabackup和其他程序协同工作。
--tables=name：正则表达式匹配database.tablename。备份匹配的表。
--tables-file=name：指定文件，一个表名一行。
--target-dir=DIRECTORY：指定backup的目的地，如果目录不存在，xtrabakcup会创建。如果目录存在且为空则成功。不会覆盖已存在的文件。
--throttle=#：指定每秒操作读写对的数量。
--tmpdir=name：当使用--print-param指定的时候打印出正确的tmpdir参数。
--to-archived-lsn=LSN：指定prepare备份时apply事务日志的LSN，只能和xtarbackup --prepare选项一起用。
--user-memory = #：通过--prepare prepare备份时候分配多大内存，目的像innodb_buffer_pool_size。默认值100M如果你有足够大的内存。1-2G是推荐值，支持各种单位(1MB,1M,1GB,1G)。
--version：打印xtrabackup版本并退出。
--xbstream：支持同时压缩和流式化。需要客服传统归档tar,cpio和其他不允许动态streaming生成的文件的限制，例如动态压缩文件，xbstream超越其他传统流式/归档格式的的优点是，并发stream多个文件并且更紧凑的数据存储（所以可以和--parallel选项选项一起使用xbstream格式进行streaming）。
```
#四、Xtrabackup 实战
##4.1 全量备份
###1.创建备份

    xtrabackup --uroot -p123456 --databases=test --backup --target-dir=/backup/xtrabackup/
如果目标目录不存在，xtrabackup 会创建它。xtrabackup不会覆盖现有文件，如果目标文件已存在它会因操作系统错误17而失败。

###2.准备备份

    xtrabackup --prepare --target-dir=/backup/xtrabackup/
一般情况下,在备份完成后，数据尚且不能用于恢复操作，因为备份的数据中可能会包含尚未提交的事务或已经提交但尚未同步至数据文件中的事务。因此，此时数据 文件仍处理不一致状态。--prepare参数实现通过回滚未提交的事务及同步已经提交的事务至数据文件使数据文件处于一致性状态。

###3.恢复备份
    systemctl stop mysqld # 关闭 MySQL 服务
    rsync -avrP /backup/xtrabackup/ /var/lib/mysql/ # 还原数据
    chown -R mysql:mysql /var/lib/mysql
    systemctl start mysqld # 重启 MySQL 服务

###4.2 增量备份
1.先创建完全备份

    xtrabackup --uroot -p123456 --databases=test --backup --target-dir=/backup/xtrabackup/
2.创建第一次增量备份

    xtrabackup --uroot -p123456 --databases=test --backup --target-dir=/backup/inc1/ --incremental-basedir=/backup/xtrabackup/
3.创建第二次增量备份

    xtrabackup --uroot -p123456 --databases=test --backup --target-dir=/backup/inc2/ --incremental-basedir=/backup/inc1/
4.准备全量备份

    xtrabackup --prepare --apply-log-only --target-dir=/backup/xtrabackup/
5.准备第一次增量备份

    xtrabackup --prepare --apply-log-only --target-dir=/backup/xtrabackup/ --incremental-dir=/backup/inc1
6.准备第二次增量备份

    xtrabackup --prepare --target-dir=/backup/xtrabackup/ --incremental-dir=/backup/inc2/
7.恢复数据

    systemctl stop mysqld # 停止服务
    rsync -avrP /backup/xtrabackup/ /var/lib/mysql/
    chown -R mysql:mysql /var/lib/mysql    
    systemctl start mysqld # 启动服务
现在数据已经恢复到执行第二次增量备份命令时的数据。