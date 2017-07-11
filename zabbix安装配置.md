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