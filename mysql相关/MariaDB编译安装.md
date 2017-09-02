MariaDB编译安装

官方下载地址: 
[https://mirrors.tuna.tsinghua.edu.cn/mariadb//mariadb-10.1.22/source/mariadb-10.1.22.tar.gz](https://mirrors.tuna.tsinghua.edu.cn/mariadb//mariadb-10.1.22/source/mariadb-10.1.22.tar.gz)

1、首先查询下是否安装了mysql或者旧版本mariadb
rpm -qa | grep mysql
删除rm -rf /etc/my.cnf

2、安装依赖包
\#  yum install  -y  libevent 
\# yum groupinstall -y Development Tools
\# yum install -y ncurses-devel openssl-devel openssl

3、创建数据库用户及组
\#groupadd mysql
\#useradd mysql -s /sbin/nologin -g mysql -M mysql

4、创建数据库数据存放目录及赋予权限
\#mkdir /appliction/mydata -p
\#chown mysql.mysql /appliction/mydata -R

一、编译安装开始
1、解压
\#tar zxf mariadb-10.1.22.tar.gz
\#cd mariadb-10.1.22
\#cmake . -DCMAKE_INSTALL_PREFIX=/appliction/mysql \      //安装目录
​          -DMYSQL_DATADIR=/appliction/mydata \      //数据库存放目录
​          -DWITH_INNOBASE_STORAGE_ENGINE=1 \       //支持数据库innobase引擎
​          -DWITH_ARCHIVE_STORAGE_ENGINE=1 \       //支持数据库archive引擎
​          -DWITH_BLACKHOLE_STORAGE_ENGINE=1 \     //支持数据库blackhole存储引擎
​          -DWITH_READLINE=1 \                                    
​          -DWITH_SSL=system \                                    
​          -DWITH_ZLIB=system \
​          -DWITH_LIBWRAP=0 \
​          -DMYSQL_UNIX_ADDR=/tmp/mysql.sock \                   
​          -DDEFAULT_CHARSET=utf8 \             //字符集utf8
​          -DDEFAULT_COLLATION=utf8_general_ci \    //校验字符
​          -DENABLED_LOCAL_INFILE=1             //允许本地导入数据

执行编译安装：

cmake . -DCMAKE_INSTALL_PREFIX=/usr/local/mysql -DMYSQL_DATADIR=/data1/mysql -DSYSCONFDIR=/etc -DWITHOUT_TOKUDB=1 -DWITH_INNOBASE_STORAGE_ENGINE=1 -DWITH_ARCHIVE_STPRAGE_ENGINE=1 -DWITH_BLACKHOLE_STORAGE_ENGINE=1 -DWIYH_READLINE=1 -DWIYH_SSL=system -DVITH_ZLIB=system -DWITH_LOBWRAP=0 -DMYSQL_UNIX_ADDR=/tmp/mysql.sock -DDEFAULT_CHARSET=utf8 -DDEFAULT_COLLATION=utf8_general_ci

这里说明一下：-DCMAKE_INSTALL_PREFIX是指定安装的位置，这里是/usr/local/mysql，-DMYSQL_DATADIR是指定MySQL的数据目录，这里是/data1/mysql，安装目录和数据目录都可以自定义设置，-DSYSCONFDIR是指定配置文件所在的目录，一般都是/etc ，具体的配置文件是/etc/my.cnf，-DWITHOUT_TOKUDB=1这个参数一般都要设置上，表示不安装tokudb引擎，tokudb是MySQL中一款开源的存储引擎，可以管理大量数据并且有一些新的特性，这些是Innodb所不具备的，这里之所以不安装，是因为一般计算机默认是没有Percona Server的，并且加载tokudb还要依赖jemalloc内存优化，一般开发中也是不用tokudb的，所以暂时屏蔽掉，否则在系统中找不到依赖会出现：CMake Error at storage/tokudb/PerconaFT/cmake_modules/TokuSetupCompilerNaNake:179 (message)这样的错误，然后后面那些参数都是可选的，可以加也可以不加，最后的编码建议设置一下，所以编译指令也可以简化成下面这样：

cmake . -DCMAKE_INSTALL_PREFIX=/usr/local/mysql -DMYSQL_DATADIR=/data1/mysql -DSYSCONFDIR=/etc -DWITHOUT_TOKUDB=1 -DMYSQL_UNIX_ADDR=/tmp/mysql.sock -DDEFAULT_CHARSET=utf8 -DDEFAULT_COLLATION=utf8_general_ci

注意：如果万一执行中有了错误，可以执行： rm -f CMakeCache.txt 删除编译缓存，让指令重新执行，否则每次读取这个文件，命令修改正确也是报错

\#make -j4
\#make install
　　cmake没问题，可以编译并且安装了： make && make install 时间有点长，耐心等待
　　执行完成也就是安装完成了，现在执行 cd /usr/local/mysql/ 进入mysql安装目录分别执行下面命令：
chown -R mysql:mysql .
scripts/mysql_install_db --datadir=/data1/mysql --user=mysql
注：可用`--defaults-file=`选择默认配置文件位置
chown -R root .
cp support-files/mysql.server /etc/init.d/mysqld
　　然后还可以将mysqld添加至系统服务：
chkconfig --add mysqld   # 添加至系统服务
chkconfig mysqld on    # 设置开机自启动
　　现在如果启动可能会报错，原因是日志目录没有建立，默认是/var/log/mariadb/mariadb.log，后来也可以修改，现在执行：
 mkdir/var/log/mariadb 建立日志目录，然后执行： /etc/init.d/mysqld start 或者 
systemctl start mysqld.service 都可以启动mysql服务
　　启动服务后，还不能马上进入mysql 
shell界面，原因是刚才编译时执行本地socket为：/tmp/mysql.sock但是查看/etc/my.cnf中配置的位置却是：/var/lib/mysql/mysql.sock，现在执行命令：
 ln -s /var/lib/mysql/mysql.sock /tmp/mysql.sock 建立软链接即可

[Ubuntu](http://www.linuxidc.com/topicnews.aspx?tid=2) 16.04 LTS 上安装 Nginx、MariaDB 和 HHVM 运行 WordPress  [http://www.linuxidc.com/Linux/2016-10/136435.htm](http://www.linuxidc.com/Linux/2016-10/136435.htm)

Ubuntu 16.04 Dockerfile 安装MariaDB  [http://www.linuxidc.com/Linux/2016-09/135260.htm](http://www.linuxidc.com/Linux/2016-09/135260.htm)

Linux系统教程：如何检查MariaDB服务端版本  [http://www.linuxidc.com/Linux/2015-08/122382.htm](http://www.linuxidc.com/Linux/2015-08/122382.htm)

Ubuntu 16.04下如何安装MariaDB  [http://www.linuxidc.com/Linux/2017-04/142915.htm](http://www.linuxidc.com/Linux/2017-04/142915.htm)

Linux下编译安装配置MariaDB数据库的方法 [http://www.linuxidc.com/Linux/2014-11/109049.htm](http://www.linuxidc.com/Linux/2014-11/109049.htm)

[CentOS](http://www.linuxidc.com/topicnews.aspx?tid=14)系统使用yum安装MariaDB数据库 [http://www.linuxidc.com/Linux/2014-11/109048.htm](http://www.linuxidc.com/Linux/2014-11/109048.htm)

安装MariaDB与MySQL并存 [http://www.linuxidc.com/Linux/2014-11/109047.htm](http://www.linuxidc.com/Linux/2014-11/109047.htm)

Ubuntu 上如何将 MySQL 5.5 数据库迁移到 MariaDB 10  [http://www.linuxidc.com/Linux/2014-11/109471.htm](http://www.linuxidc.com/Linux/2014-11/109471.htm)

**[翻译]Ubuntu 14.04 (Trusty) Server 安装 MariaDB**  [http://www.linuxidc.com/Linux/2014-12/110048htm](http://www.linuxidc.com/Linux/2014-12/110048htm)

Ubuntu 14.04(Trusty)安装MariaDB 10数据库  [http://www.linuxidc.com/Linux/2016-11/136833.htm](http://www.linuxidc.com/Linux/2016-11/136833.htm)



注：mysql启动脚本加载配置文件my.cnf 优先级（存在的情况下） /etc/my.cnf > basedir/my.cnf > datadir/my.cnf  。不存在my.cnf文件的情况下会根据初始化信息自动加载一个3306端口的配置文件