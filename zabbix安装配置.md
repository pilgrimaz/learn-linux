1. 安装zabbix
 安装rpm包的lamp环境(yum安装)
 ```
 yum install -y httpd mysql mysql-libs php php-mysql mysql-server php-bcmath php-gd php-mbstring
 ```
 安装zabbix服务端：
 ```
yum install zabbix20 zabbix20-agent zabbix20-server zabbix20-server-mysql zabbix20-web zabbix20-web-mysql net-snmp-devel
/etc/init.d/zabbix-server start;/etc/init.d/zabbix-agent start
/etc/init.d/httpd start;/etc/init.d/mysqld start
mysql -uroot -p -e "create database zabbix"
mysql -uroot -p --default-character-set=utf8 zabbix < /usr/share/zabbix-mysql/schema.sql
mysql -uroot -p --default-character-set=utf8 zabbix < /usr/share/zabbix-mysql/images.sql
mysql -uroot -p --default-character-set=utf8 zabbix < /usr/share/zabbix-mysql/data.sql
```
2. 网页安装zabbix
浏览器访问 http://ip/zabbix，默认会有“It is not safe to rely on the system's timezone setting"这样的警告信息，需要vim /etc/php.ini 设置date.timezone='Asia/Chongqing'
点next，解决相关的报错信息（vim /etc/php.ini）
点netx，输入mysql相关信息，（此处可建立一个zabbix的mysql用户，`mysql -uroot -e "grant all on zabbix.* to 'zabbix'@localhost identified by 'zabbixpasswd'"`,建立用户后需要到/etc/zabbix_server.conf添加数据库用户名和密码）首先要测试一下，如果不通过，则需要调试，测试通过后，
点next，Name可自定义一个，点next，再点next，最后finish
默认管理员账号为admin：zabbix
这是如果遇到"zabbix server is not running"这样的错误，需要编辑一下/etc/zabbix/zabbix_server.conf,配置DBUser，DBPassword（跟上面一样）
3. 接入要监控的主机
在客户端上 yum install zabbix20-agent
vim /etc/zabbix_agent.conf //更改Server=服务端ip；ServerActive=0.0.0.0:10050；Hostname=aming（自定义，但要唯一）
启动客户端/etc/init.d/zabbix-agent start
服务端上命令行测试：zabbix_get -s 客户端ip -p10050 -k "system.hostname"
在web界面下，点"configuration"-->"host"-->右上角点"Create Host"其中host name，visible name自定义，可以选择greoups，这里默认即可，ip address写入客户端ip
配置监控项目模板：点"temolates",点add，在弹出的小窗口中选择Template OS Linux,然后点select，最后点save
4. 自定义templates
Zabbix自带了很多模板，模板中有很多监控项目，比如CPU、网卡、内存、进程等。使用系统自带模板有点太多了，所以我们可以自定义模板。点configuration选择templates，点save
然后我们去挑选一些项目拷贝到该模板下：比如我们找到template OS Linux点一下items，选择我们想要的项目，然后在下面选中copy selected to ...然后点go