#### centos安装cacti

一、 安装 cacti服务端

1. 首先要安装epel扩展源
   rpm -ivh  http://www.aminglinux.com/bbs/da ... ease-6-7.noarch.rpm

2. （lamp）然后分别安装httpd、php、mysql
   yum install -y  httpd php php-mysql mysql mysql-server mysql-devel php-gd  libjpeg libjpeg-devel libpng libpng-devel

3. 安装cacti  net-snmp  rrdtool
   yum install -y cacti  net-snmp  net-snmp-utils  rrdtool

4. 启动服务：
   /etc/init.d/mysqld start
   /etc/init.d/httpd  start
   /etc/init.d/snmpd start

5. 编辑httpd配置文件
   vim /etc/httpd/conf.d/cacti.conf  
   把"Deny from all" 改为  "Allow from all"
   /etc/init.d/httpd  restart

6. 导入数据创建cacti库
   mysql -uroot  -e "create database cacti"

创建cacti用户
mysql -uroot -e "grant all on cacti.* to 'cacti'@'localhost' identified by 'cacti';"

导入sql文件
mysql -uroot cacti < /usr/share/doc/cacti-0.8.8b/cacti.sql

7.  编辑cacti配置文件
    vim /usr/share/cacti/include/config.php  更改如下：
    $database_type = "mysql";
    $database_default = "cacti";
    $database_hostname = "localhost";
    $database_username = "cacti";
    $database_password = "cacti";
    $database_port = "3306";
    $database_ssl = false;


8. web访问cacti并安装
   http://ip/cacti/  
   点两下“next” 和一次”Finish“ 即可
   输入admin   admin 登录，重新设置新的密码

9. 执行poller.php, 生成图形， 加入计划任务
   /usr/bin/php /usr/share/cacti/poller.php添加cron任务
   cront -e  增加：
   */5 * * * *  /usr/bin/php /usr/share/cacti/poller.php

二、 安装客户端（增加一个linux服务器）
1. 安装snmp
   yum install -y net-snmp

2. 修改snmpd.conf
   修改syslocation以及syscontact, 其中syslocation 可以写本机ip，syscontact写管理员邮箱

syslocation 11.20
syscontact Root 1212@1212.com

3. 启动snmp
   service  snmpd  start

4. 登录cacti管理后台，点console , 再点Device， 在右上角点”Add“
   Description  写本机ip或你自定义一个名字
   Hostname  写本机ip
   Host Template  选ucd/net  SNMP Host
   SNMP Version  选Version 2
   点右下角的create
   点右上角的”Create Graphs for this Host“
   Graph Types:  选择SNMP - Interface Statistics
   在下面框中选择要监控的网卡，比如我选择eth0, 在最右侧小方块里打对勾，然后点右下角的create
   Graph Types:  再选择 Graph Template Based
   在下面的框中，选择你要监控的项目，比如ucd/net - Load Average
   在右侧小方块中打对勾，然后点右下角的create


5. 点左侧的Graph Trees
   选中”Default Tree“
   点右上角的Add
   Tree Item Type 选择 ”Host“
   Host 选择我们刚刚增加的那个机器ip
   点右下角的create

6. 点左上角的Graphs
   在左侧可以看到
   Defaut Tree下面已经增加了我们刚刚添加的主机，图形一开始不会那么快出来，要等一小会才可以。

