linux学习笔记

## 一、目录含义

usr是曾经的用户目录，现在已经被/home目录取代了。可以理解为Unix System Resource，即Unix系统资源目录，系统核心所在，现代的/usr只专门存放各类程序和数据。

/usr：系统级的目录，可以理解为C:/windows

/usr/lib理解为C:/windows/System32

/usr/local：理解为C:/windows/Program Files。用户自己编译的软件会默认安装到这个目录下

/opt:用户级的程序目录。可以理解为D:/software。这里可以放置第三方大型软件，当不需要了时，直接rm-rf。麒麟系统，默认第三方软件安装目录示例：/opt/software/java/jdk1.8。当磁盘空间不足时，可将/opt单独挂载到其他磁盘使用



## 二、常用命令

1、删除目录和文件：`rm [options] name...`

命令参数：

`-i` 删除前逐一询问确认

`-f `**文档类型**直接删除，无需逐一确认，即使原档案属性设为唯读

`-r` 将目录及以下全部删除

示例：目录直接删除 `*[root@localhost opt]# rm -rf test2* `

2、切换root用户

直接输入`su`，回车，输入密码即可。若失败，可能没有为root设置密码，使用`sudo password root`来设置root账号密码

su含义为switch user，用于变更用户。sudo命令则是以管理员身份执行指令

3、常用单个命令

| 命令                     | 含义         | 命令 | 含义 |
| ------------------------ | :----------- | ---- | ---- |
| whoami                   | 查看当前用户 |      |      |
| shutdown -r              | 重新启动     |      |      |
| ip address               | 查看ip       |      |      |
| sudo -i                  | 切换root用户 |      |      |
| netstat -anp \|grep 8080 |              |      |      |
| find -name 'nginx'       | 查找         |      |      |
|                          |              |      |      |
|                          |              |      |      |
|                          |              |      |      |





当前环境：centos7及以上

当前用户：root

node、jdk默认安装目录`/usr/local/lib

## 1、安装node

node官网下载node-v14.9.0-linux-x64.tar.gz文件

xftp上传到`/usr/local/lib`目录下，解压文件，当前目录下命令`tar -xzvf node-v14.9.0-linux-x64.tar.gz`。

将目录改为node14.9.0，命令`mv node-v14.9.0-linux-x64 node14.9.0`

1、验证是否安装成功。

`cd node14.9.0/bin `

`cd ./node -v`，输出v14.9.0成功

2、配置环境变量，满足全局命令可用

输入命令`vim /etc/profile`，打开文件进行编辑,文档尾部加入，多个环境变量用:进行隔开即可

```bash
export NODE_HOME=/usr/local/lib/node14.9.0
export PATH=${NODE_HOME}/BIN:$PATH
```

键盘Esc按键，推出编辑模式，shift+:保存编辑内容

刷新|重启电脑生效。刷新命令`source /etc/profile`，如果不是可执行命令，则需要重启电脑

## 2、安装JRE

将压缩包`jre-8u351-linux-x64.tar.gz`放到`/usr/local/lib`目录下

`tar -xzvf jre-8u351-linux-x64.tar.gz`解压文件，修改目录名为jre1.8

编辑`etc/profile`文件。在export处添加如下

```bash
export JRE_HOME=/usr/local/lib/jre1.8
export PATH=${JRE_HOME}/bin:${NODE_HOME}/bin:$PATH
```

配置环境变量、刷新

`source /etc/profile`

## 3、Mysql

(https://www.cnblogs.com/renlywen/p/13502794.html)

1、上传文件到`/usr/local`解压文件，目录更名为mysql5.7

2、 给mysql目录添加权限，通常来说是直接创建mysql用户然后，添加权限，不过我自己觉得麻烦是直接给mysql目录用root账户

 ` chown -R root:root /usr/local/mysql`

3、 在mysql5.7目录中创data和logs目录 

` mkdir {data,logs} `

4、 修改|新增/etc/my.cnf文件，有的系统可能还要**创建/etc/my.cnf.d文件夹**否则会报错找不到该目录 

```bash
[root@master mysql]# chown 777 /etc/my.cnf
[root@master mysql]# vim /etc/my.cnf

[mysqld]
character_set_server=utf8
character_set_server=utf8 
init_connect='SET NAMES utf8' 
basedir=/root/datasoft/mysql 
datadir=/root/datasoft/mysql/data 
socket=/root/datasoft/mysql/mysql.sock
#开启ip绑定 
#bind-address = 0.0.0.0 
log_timestamps = SYSTEM 
#open_files_limit=30000 max_connections=3000 
#控制其通信缓冲区的最大长度 
max_allowed_packet=256M
#设置忽略大小写(简单来说就是sql语句是否严格)，默认库名表名保存为小写, 不区分大小写
lower_case_table_names = 1 
#Disabling symbolic-links is recommended to prevent assorted security risks 
symbolic-links=0
[mysqld_safe] 
log-error=/root/datasoft/mysql/logs/mysqld.log 
pid-file=/root/datasoft/mysql/data/mysqld.pid

[client] 
socket=/root/datasoft/mysql/mysql.sock 
#default-character-set=utf8
# include all files from the config directory
!includedir /etc/my.cnf.d

```

备份

```bash
[mysqld]
#设置mysql的安装目录
basedir =/usr/local/mysql-5.7.37
#设置mysql数据库的数据存放目录
datadir = /usr/local/mysql-5.7.37/data
#设置端口
port = 3306

socket = /tmp/mysql.sock
#设置字符集
character-set-server=utf8
#日志存放目录
log-error = /usr/local/mysql-5.7.37/data/mysqld.log
pid-file = /usr/local/mysql-5.7.37/data/mysqld.pid
#允许时间类型的数据为零(去掉NO_ZERO_IN_DATE,NO_ZERO_DATE)
sql_mode=ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
#ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION

```

5、初始化mysql

 basedir为我们的mysql目录，datadir为我们创建的保存数据的目录。初始化后会打印出初始密码，如果忘记了可以清空datadir目录重新执行初始化。 

查看日志：`cat /usr/local/mysql-5.7.37/logs/mysqld.log`。模拟时并未打印密码也没有生成日志

`[root@master mysql]# ./bin/mysqld --initialize --user=root --basedir=/root/datasoft/mysql/ --datadir=/root/datasoft/mysql/data/`

6、把启动脚本放到开机初始化目录

`cp support-files/mysql.server /etc/init.d/mysql`

7、开启mysql

`service mysql start`

8、登录，密文

`mysql -uroot -p`

9、修改初始密码

`mysql>set password for root@localhost = password('123456');`

10、添加远程访问权限

`mysql>GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'mysql密码' WITH GRANT OPTION;`

11、刷新

mysql>FLUSH PRIVILEGES;
mysql>exit;

12、打开防火墙端口

`firewall-cmd --zone=public --add-port=3306/tcp --permanent`

13、重启防火墙

firewall-cmd --reload        #重启firewall
firewall-cmd --list-ports    #查看已经开放的端口

14、设置开机启动

```bash
cp -a /usr/local/mysql/support-files/mysql.server /etc/init.d/mysql
查看mysql服务是否在服务配置中
chkconfig --list mysql
若没有，则把mysql注册为开机启动的服务，然后在进行查看
chkconfig --add mysql
chkconfig --list mysql

启动 或 停止
service mysql start
service mysql stop


```

## mysql卸载

查看安装了哪写服务

`rpm -qa|grep -i mysql`



## 4、redis

1、解压文件，可不用修改目录名。进入redis-5.0.4目录下，执行`make`

(可选make install PREFIX=/usr/local/lib/redis-5.0.4)库文件会存放在/usr/local/lib目录。配置文件会存放在/usr/local/etc目录。其他的资源文件会存放在usr/local/share目录。这里指定号目录也方便后续的卸载，后续直接rm -rf /usr/local/redis 即可删除redis。 这一步执行完以后，在/opt/redis-5.0.4目录下就会出现一个bin目录，里面是redis的一些可执行程序，主要是与源码分开 

2、make完后 redis-5.0.4/src目录下会出现编译后的redis服务程序redis-server,还有用于测试的客户端程序redis-cli 

3、进入redis-5.0.4/src目录下，启动默认服务，输入` ./redis-server & ` &后台运行。

 注意这种方式启动redis 使用的是默认配置。也可以通过启动参数告诉redis使用指定配置文件使用下面命令启动 

` ./src/redis-server& ./redis.conf `

> 设置开机自启

## 5、ngnix

> [https://blog.csdn.net/xiawenquan/article/details/125122794](https://blog.csdn.net/xiawenquan/article/details/125122794)



## 6、jar开机自启

> https://blog.csdn.net/lep1995/article/details/129176336