[TOC]



### LAMP安装

#### 1. 安装mysql####

1. 下载mysql到/usr/local/src/
```shell
cd /usr/local/src/
wget http://syslab.comsenz.com/downloads/linux/mysql-5.1.40-linux-i686-icc-glibc23.tar.gz
```
2. 解压
```shell
[root@localhost src]# tar zxvf /usr/local/src/mysql-5.1.40-linux-i686-icc-glibc23.tar.gz
```
3. 把解压完的数据移动到/usr/local/mysql
```shell
[root@localhost src]# mv mysql-5.1.40-linux-i686-icc-glibc23 /usr/local/mysql
```
4. 建立mysql用户
```shell
[root@localhost src]# useradd -s /sbin/nologin mysql
```
5. 编译安装数据库

   ```shell
   [root@localhost mysql-5.1.55]# ./configure --prefix=/usr/local/mysql 
   --with-charset=utf8 --with-collation=utf8_general_ci 
   --with-extra-charsets=gbk,gb2312
   ```
   这这里呢 我添加的参数比较简单 可以根据需要添加以下参数

   --prefix=/usr/local/mysql //MySQL安装目录 
   --datadir=/mydata //数据库存放目录 
   --with-extra-charsets=gb2312 \添加gb2312中文字符支持 
   --with-charset=utf8 //使用UTF8格式 
   --without-debug \去除debug模式 
   --with-extra-charsets=complex //安装所有的扩展字符集 
   --enable-thread-safe-client //启用客户端安全线程 
   --with-big-tables //启用大表 
   --with-ssl //使用SSL加密 
   --with-embedded-server //编译成embedded MySQL library (libmysqld.a), 
   --enable-local-infile //允许从本地导入数据 
   --enable-assembler //汇编x86的普通操作符，可以提高性能 
   --with-plugins=innobase //数据库插件 
   --with-plugins=partition //分表功能，将一个大表分割成多个小表

6. 初始化数据库
```shell
[root@localhost src]# cd /usr/local/mysql
[root@localhost mysql]# mkdir -p /data/mysql ; chown -R mysql:mysql /data/mysql
[root@localhost mysql]# ./scripts/mysql_install_db --user=mysql --datadir=/data/mysql
```
`--user` 定义数据库的所属主， `--datadir` 定义数据库安装到哪里，建议放到大空间的分区上，这个目录需要自行创建。这一步骤很关键，如果你看到两个 “OK” 说明执行正确。
7. 拷贝配置文件
```shell
[root@localhost mysql]# cp support-files/my-large.cnf /etc/my.cnf
```
8. 拷贝启动脚本文件并修改其属性
```
[root@localhost mysql]# cp support-files/mysql.server  /etc/init.d/mysqld
[root@localhost mysql]# chmod 755 /etc/init.d/mysqld
```
9. 修改启动脚本
```
[root@localhost mysql]# vim /etc/init.d/mysqld
```
需要修改的地方有 “datadir=/data/mysql” （前面初始化数据库时定义的目录）
10. 把启动脚本加入系统服务项，并设定开机启动，启动mysql
```
[root@localhost mysql]# chkconfig --add mysqld
[root@localhost mysql]# chkconfig mysqld on
[root@localhost mysql]# service mysqld start
```
如果启动不了，请到 /data/mysql/ 下查看错误日志，这个日志通常是主机名.err. 检查mysql是否启动的命令为:
```
[root@localhost mysql]# ps aux |grep mysqld
```
#### 2.  安装Apache####

同样apache也需要到官网下载合适的版本，目前使用较多的版本为2.0或者2.2阿铭建议下载2.2版本。apache官网下载地址： [http://www.apache.org/dyn/closer.cgi](http://www.apache.org/dyn/closer.cgi) 你也可以使用阿铭提供的地址下载。

```
[root@localhost mysql]# cd /usr/local/src/
[root@localhost src]# wget http://syslab.comsenz.com/downloads/linux/httpd-2.2.16.tar.gz

```

解压:

```
[root@localhost src]# tar zxvf httpd-2.2.16.tar.gz
```

配置编译参数:

```
[root@localhost src]# cd httpd-2.2.16
[root@localhost httpd-2.2.16]# ./configure \
--prefix=/usr/local/apache2 \
--with-included-apr \
--enable-so \
--enable-deflate=shared \
--enable-expires=shared \
--enable-rewrite=shared \
--with-pcre
```

`--prefix` 指定安装到哪里， `--enable-so` 表示启用DSO [[1\]](http://www.apelearn.com/study_v2/chapter17.html#id5) `--enable-deflate=shared` 表示共享的方式编译deflate，后面的参数同理。如果这一步你出现了这样的错误:

```
error: mod_deflate has been requested but can not be built due to prerequisite failures
```

解决办法是:

```
yum install -y zlib-devel
```

为了避免在make的时候出现错误，所以最好是提前先安装好一些库文件:

```
yum install -y pcre pcre-devel apr apr-devel
```

编译:

```
[root@localhost httpd-2.2.16]# make
```

安装:

```
[root@localhost httpd-2.2.16]# make install
```

以上两个步骤都可以使用 `echo $?` 来检查是否正确执行，否则需要根据错误提示去解决问题。

#### 3. 安装PHP####

阿铭写这本教程时，php当前最新版本为5.5, 相信大多网站还在跑着5.2甚至更老的版本，其实5.2版本的php很经典也很稳定，因为阿铭的公司一直在使用5.2版本，但是考虑到版本太老，难免会有些漏洞，所以建议你使用5.3或者5.4版本，php官方下载地址: [http://www.php.net/downloads.php](http://www.php.net/downloads.php)

下载php:

```
[rot@localhost httpd-2.2.16]# cd /usr/local/src
[root@localhost src]# wget http://am1.php.net/distributions/php-5.3.27.tar.gz
```

解压:

```
[root@localhost src]# tar zxf php-5.3.27.tar.gz
```

配置编译参数:

```
[root@localhost src]# cd php-5.3.27
[root@localhost php-5.3.27]# ./configure \
--prefix=/usr/local/php \
--with-apxs2=/usr/local/apache2/bin/apxs \
--with-config-file-path=/usr/local/php/etc  \
--with-mysql=/usr/local/mysql \
--with-libxml-dir \
--with-gd \
--with-jpeg-dir \
--with-png-dir \
--with-freetype-dir \
--with-iconv-dir \
--with-zlib-dir \
--with-bz2 \
--with-openssl \
--with-mcrypt \
--enable-soap \
--enable-gd-native-ttf \
--enable-mbstring \
--enable-sockets \
--enable-exif \
--disable-ipv6
```

在这一步，阿铭遇到如下错误:

```
configure: error: xml2-config not found. Please check your libxml2 installation.

```

解决办法是:

```
yum install -y libxml2-devel

```

还有错误:

```
configure: error: Cannot find OpenSSL's <evp.h>

```

解决办法是:

```
yum install -y openssl openssl-devel

```

错误:

```
checking for BZip2 in default path... not found
configure: error: Please reinstall the BZip2 distribution

```

解决办法:

```
yum install -y bzip2 bzip2-devel

```

错误:

```
configure: error: png.h not found.

```

解决办法:

```
yum install -y libpng libpng-devel

```

错误:

```
configure: error: freetype.h not found.

```

解决办法:

```
yum install -y freetype freetype-devel
```

错误:

```
configure: error: mcrypt.h not found. Please reinstall libmcrypt.
```

解决办法:

```
rpm -ivh "http://www.aminglinux.com/bbs/data/attachment/forum/month_1211/epel-release-6-7.noarch.rpm"
yum install -y  libmcrypt-devel
```

因为centos6.x 默认的yum源没有libmcrypt-devel 这个包，只能借助第三方yum源。

编译:

```
[root@localhost php-5.3.27]# make

```

在这一步，你也许还会遇到诸多错误，没有关系，请仔细查看报错信息，解决办法很简单，就是装缺少的库。你可以把错误信息复制到google上搜一下，如果实在是解决不了，请到阿铭论坛([http://www.aminglinux.com/bbs/forum-40-1.html](http://www.aminglinux.com/bbs/forum-40-1.html))发帖请教阿铭吧。

安装:

```
[root@localhost php-5.3.27]# make install
```

拷贝配置文件:

```
[root@localhost php-5.3.27]# cp php.ini-production /usr/local/php/etc/php.ini
```

#### 4. apache结合php####

Apache主配置文件为：/usr/local/apache2/conf/httpd.conf

```
vim /usr/local/apache2/conf/httpd.conf
```

找到:

```
AddType application/x-gzip .gz .tgz
```

在该行下面添加:

```
AddType application/x-httpd-php .php
```

找到:

```
<IfModule dir_module>
    DirectoryIndex index.html
</IfModule>
```

将该行改为:

```
<IfModule dir_module>
    DirectoryIndex index.html index.htm index.php
</IfModule>
```

找到:

```
#ServerName www.example.com:80

```

修改为:

```
ServerName localhost:80
```

#### 5. 测试LAMP是否成功####

启动apache之前先检验配置文件是否正确:

```
/usr/local/apache2/bin/apachectl -t

```

如果有错误，请继续修改httpd.conf, 如果是正确的则显示为 “Syntax OK”, 启动apache的命令为:

```
/usr/local/apache2/bin/apachectl start
```

查看是否启动:

```
[root@localhost ~]# netstat -lnp |grep httpd
tcp        0      0 :::80                       :::*   LISTEN      7667/httpd
```

如果有显示这行，则启动了。 也可以使用curl命令简单测试:

```
[root@localhost ~]# curl localhost
<html><body><h1>It works!</h1></body></html>
```

只有显示这样才正确。

测试是否正确解析php:

```
vim /usr/local/apache2/htdocs/1.php
```

写入:

```
<?php
    echo "php解析正常";
?>
```

保存后，继续测试:

```
curl localhost/1.php
```

看是否能看到如下信息:

```
[root@localhost ~]# curl localhost/1.php
php解析正常[root@localhost ~]#
```

只有显示如阿铭这样才正确。

初次使用浏览器访问我们的web服务的时候，你可能无法访问，这是因为防火墙的缘故。请运行下面的命令:

```
[root@localhost ~]# iptables -F
```

这样就可以清除系统默认的防火墙规则，放行80端口。





