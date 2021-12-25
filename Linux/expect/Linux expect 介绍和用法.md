expect是一个自动化交互套件，主要应用于执行命令和程序时，系统以交互形式要求输入指定字符串，实现交互通信。

expect自动交互流程：

spawn启动指定进程---expect获取指定关键字---send向指定程序发送指定字符---执行完成退出.

注意该脚本能够执行的前提是安装了expect

    yum install -y expect

expect常用命令总结:
````
spawn               交互程序开始后面跟命令或者指定程序
expect              获取匹配信息匹配成功则执行expect后面的程序动作
send exp_send       用于发送指定的字符串信息
exp_continue        在expect中多次匹配就需要用到
send_user           用来打印输出 相当于shell中的echo
exit                退出expect脚本
eof                 expect执行结束 退出
set                 定义变量
puts                输出变量
set timeout         设置超时时间
````
示例：

1.ssh登录远程主机执行命令,执行方法 expect 1.sh 或者 ./1.sh
````
#!/usr/bin/expect
spawn ssh saneri@192.168.56.103 df -Th
expect "*password"
send "123456\n"
expect eof
````

2. ssh远程登录主机执行命令，在shell脚本中执行expect命令,执行方法sh 2.sh、bash 2.sh 或./2.sh都可以执行.
````
#!/bin/bash

passwd='123456'

/usr/bin/expect <<-EOF

set time 30
spawn ssh saneri@192.168.56.103 df -Th
expect {
"*yes/no" { send "yes\r"; exp_continue }
"*password:" { send "$passwd\r" }
}
expect eof
EOF
````

3.expect执行多条命令
````
#!/usr/bin/expect -f

set timeout 10

spawn sudo su - root
expect "*password*"
send "123456\r"
expect "#*"
send "ls\r"
expect "#*"
send "df -Th\r"
send "exit\r"
expect eof
````
4. 创建ssh key，将id_rsa和id_rsa.pub文件分发到各台主机上面。
````
1.创建主机配置文件

[root@localhost script]# cat host 
192.168.1.10 root 123456
192.168.1.20 root 123456
192.168.1.30 root 123456

[root@localhost script]# ls
copykey.sh  hosts
2.编写copykey.sh脚本,自动生成密钥并分发key.
[root@localhost script]# vim copykey.sh

#!/bin/bash

# 判断id_rsa密钥文件是否存在
if [ ! -f ~/.ssh/id_rsa ];then
 ssh-keygen -t rsa -P "" -f ~/.ssh/id_rsa
else
 echo "id_rsa has created ..."
fi

#分发到各个节点,这里分发到host文件中的主机中.
while read line
  do
    user=`echo $line | cut -d " " -f 2`
    ip=`echo $line | cut -d " " -f 1`
    passwd=`echo $line | cut -d " " -f 3`
    
    expect <<EOF
      set timeout 10
      spawn ssh-copy-id $user@$ip
      expect {
        "yes/no" { send "yes\n";exp_continue }
        "password" { send "$passwd\n" }
      }
     expect "password" { send "$passwd\n" }
EOF
  done <  hosts
````
5. shell调用expect执行多行命令. 
````
#!/bin/bash 
ip=$1  
user=$2 
password=$3 

expect <<EOF  
    set timeout 10 
    spawn ssh $user@$ip 
    expect { 
        "yes/no" { send "yes\n";exp_continue } 
        "password" { send "$password\n" }
    } 
    expect "]#" { send "useradd hehe\n" } 
    expect "]#" { send "touch /tmp/test.txt\n" } 
    expect "]#" { send "exit\n" } expect eof 
 EOF  
 #./ssh5.sh 192.168.1.10 root 123456
````
6.使用普通用户登录远程主机，并通过sudo到root权限，通过for循环批量在远程主机执行命令.
````
$ cat timeout_login.txt 
10.0.1.8
10.0.1.34
10.0.1.88
10.0.1.76
10.0.1.2
10.0.1.3

#!/bin/bash

for i in `cat /home/admin/timeout_login.txt`
do

    /usr/bin/expect << EOF
    spawn /usr/bin/ssh -t -p 22022 admin@$i "sudo su -"

    expect {
        "yes/no" { send "yes\r" }
    }   

    expect {
        "password:" { send "xxo1#qaz\r" }
    }
    
    expect {
        "*password*:" { send "xx1#qaz\r" }
    }

    expect "*]#"
    send "df -Th\r"
    expect "*]#"
    send "exit\r"
    expect eof

EOF
done
````
密码过期需要批量修改密码
````
#!/bin/bash

for i in `cat /root/soft/ip.txt`
do

    /usr/bin/expect << EOF
    spawn /usr/bin/ssh root@$i

    expect {
        "UNIX password" { send "Huawei@123\r" }
    }
    
    expect {
        "New password:" { send "xxHuzzawexxi@1234#\r" }
    }

   expect {
        "Retype new password:" { send "xxHuzzawexxi@1234#\r" }
    }

    expect "*]#"
    send "echo Huawei@123|passwd --stdin root\r"
    expect "*]#"
    send "exit\r"
    expect eof
EOF
done
````