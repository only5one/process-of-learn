#一、mysqldump 简介
mysqldump 是 MySQL 自带的逻辑备份工具。

它的备份原理是通过协议连接到 MySQL 数据库，将需要备份的数据查询出来，将查询出的数据转换成对应的insert 语句，当我们需要还原这些数据时，只要执行这些 insert 语句，即可将对应的数据还原。

#二、备份命令
##2.1 命令格式
    mysqldump [选项] 数据库名 [表名] > 脚本名
或
    
    mysqldump [选项] --数据库名 [选项 表名] > 脚本名
或
    
    mysqldump [选项] --all-databases [选项]  > 脚本名
##2.2 选项说明
![img_1.png](../../day06/images/img_1.png)
##2.3 实例
备份所有数据库：

    mysqldump -uroot -p --all-databases > /backup/mysqldump/all.db
备份指定数据库：

    mysqldump -uroot -p test > /backup/mysqldump/test.db
备份指定数据库指定表(多个表以空格间隔)

    mysqldump -uroot -p  mysql db event > /backup/mysqldump/2table.db
备份指定数据库排除某些表

    mysqldump -uroot -p test --ignore-table=test.t1 --ignore-table=test.t2 > /backup/mysqldump/test2.db

#三、还原命令
##3.1 系统行命令
    mysqladmin -uroot -p create db_name
    mysql -uroot -p  db_name < /backup/mysqldump/db_name.db
注：在导入备份数据库前，db_name如果没有，是需要创建的； 而且与db_name.db中数据库名是一样的才可以导入。
##3.2 soure 方法
    mysql > use db_name
    mysql > source /backup/mysqldump/db_name.db