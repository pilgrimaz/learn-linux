[](PHP添加编译时忘添加的模块方式.md)1. 安装zabbix
 安装rpm包的lamp环境(yum安装)
 ```
  yum install -y httpd mysql mysql-libs php php-mysql mysql-server php-bcmath php-gd php-mbstring
 ```
 安装zabbix服务端：
 ```
yum install zabbix20 zabbix20-agent zabbix20-server zabbix20-server-mysql     zabbix20-web zabbix20-web-mysql net-snmp-devel
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
点netx，输入mysql相关信息，（此处可建立一个zabbix的mysql用户，`mysql -uroot -e "grant all on zabbix.* to 'zabbix'@localhost identified by 'zabbixpasswd'"`,
**建立用户后需要到/etc/zabbix_server.conf添加数据库用户名和密码）首先要测试一下，如果不通过，则需要调试，**测试通过后，
点next，Name可自定义一个，点next，再点next，最后finish
默认管理员账号为admin：zabbix
**这时如果遇到"zabbix server is not running"这样的错误，需要编辑一下/etc/zabbix/zabbix_server.conf,配置DBUser，DBPassword（跟上面一样）**
3. 接入要监控的主机
在客户端上 yum install zabbix20-agent
vim /etc/zabbix_agent.conf //更改Server=服务端ip；ServerActive=0.0.0.0:10050；Hostname=aming（自定义，但要唯一）
启动客户端/etc/init.d/zabbix-agent start
服务端上命令行测试：zabbix_get -s 客户端ip -p10050 -k "system.hostname"</br>
在web界面下，点"configuration"-->"host"-->右上角点"Create Host"其中host name，visible name自定义，可以选择greoups，这里默认即可，ip address写入客户端ip
配置监控项目模板：点"temolates",点add，在弹出的小窗口中选择Template OS Linux,然后点select，最后点save
4. 自定义templates
Zabbix自带了很多模板，模板中有很多监控项目，比如CPU、网卡、内存、进程等。使用系统自带模板有点太多了，所以我们可以自定义模板。点configuration选择templates，点save
然后我们去挑选一些项目拷贝到该模板下：比如我们找到template OS Linux点一下items，选择我们想要的项目，然后在下面选中copy selected to ...然后点go
Group选中templates，找到刚才我们自定义的templates，点copy
点configuration选择templates可以看到新建的templates中已经有刚刚我们copy的items了
我们可以使用和上面相同的方法自定义拷贝triggers（触发器），它用来设置告警的阀值，当然我们也可以自定义编辑它
5. 配置发邮件
yum install -y sendmail;
mkdir -p /home/zabbix/bin
vim /home/zabbix/bin/baojing.sh  //内容：
```shell
#!/bin/bash
echo "$3" |/bin/mail -s "$2" $1
chmod +x /home/zabbix/bin/baojing.sh
```
在zabbix_server.conf配置文件中，有参数AlertScriptsPath和ExternalScripts
AlertScriptsPath=/home/zabbix/bin/ --用户自定义的media types脚本
ExternalScripts=/home/zabbix/bin/ --用户自定义的检查的脚本（item）
这样才能找到你的脚本，因为你在fronted中只是输入脚本的名称，没有路径。

6. 创建mediea types:"Administration"-->"Media types",点击右上角"Create Media Type"其中Description填"baojing"或其它自定义名称，Type选择"Script",Script填"baojing.sh"然后点"Save"。
创建user:"administration"-->"users"在右上角，选择"Users",点击"Create User".alias:test1,自定义name和lastname
password:123456;group选择guest，回到上面点一下media,type选择baojing，send to写要发送邮件的邮箱，点add，最后点save
创建action:"configuration"-->actions,右上角"Create Actions"Name自定义，我这里写"baojing"，其他默认，然后点右侧的"Operations"下的"New"按钮，"Operation Type"选择"send message","Send Message to"选择一个或多个要发送消息的用户组，send to Users选择我们之前新增的test1，"send only to"选择baojing，点一下add最后点save





-----------

注1:Zabbix修改中文及中文乱码解决办法
 Zabbix 默认为英文界面，可能很多英文不好的朋友们不太习惯使用，下面介绍中文汉化方法，其实很简单：
点选 Chinese(zh_CN)即可；
汉化完成后，可能会出现两种乱码：

1、历史记录处出现 ???? 乱码：
出现原因：
mysql数据库默认字符集为 latin1，而 zabbix 需要使用 utf8，在初始化创建 zabbix 库时没有指定具体的字符集，倒入三张表时会继承 Mysql 的默认字符集，所以此处会出现乱码；

解决办法：
1、将 zabbix 数据库中的表备份；
2、手动删除 zabbix 数据库；
3、重新创建 zabbix 库时手动指定字符集为 utf8；
  `create database zabbix default charset utf8`
4、将倒出的 sql 文件中字符集为latin1的表字符集替换为 utf8；
5、将备份的zabbix库重新倒入即可；

此时重新访问 zabbix web页面，点击几次菜单，历史记录处一切正常；

2、graphs、Green 菜单下出现方框文字乱码：

出现原因：

由于zabbix的web端没有中文字库，我们需要把中文字库加上即可；

解决办法：

下载中文字体：

wget http://down1.chinaunix.net/distfiles/ttf-arphic-uming_0.0.20050501-1.tar.gz
tar xf /root/ttf-arphic-uming_0.0.20050501-1.tar.gz
cd /usr/local/apache/htdocs/zabbix/fonts  ## 注意此处为zabbix web文件所在路径
mv DejaVuSans.ttf /root/        ## 备份原有字体文件
cp /root/ttf-arphic-uming_0.0.20050501/uming.ttf  ./DejaVusans.ttf # 将下载的字体替换到此处；
一切正常

-----------

注2：zabbix安装过程中php缺少动态扩展解决办法：参见
(PHP添加编译时忘添加的模块方式.md)
安装msqli是要注明mysqli位置 编译时要用 
```
configure：# ./configure --with-php-config=/usr/local/php/bin/php-config --with-mysqli=/usr/local/MySQL/bin/mysql_config
```
（/usr/local/MYSQL 为mysql的安装目录）
