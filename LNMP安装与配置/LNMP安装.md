
```
cd /usr/local/src/
wget http://syslab.comsenz.com/downloads/linux/mysql-5.1.40-linux-i686-icc-glibc23.tar.gz

```

1. 解压

```
[root@localhost src]# tar zxvf /usr/local/src/mysql-5.1.40-linux-i686-icc-glibc23.tar.gz

```

1. 把解压完的数据移动到/usr/local/mysql

```
[root@localhost src]# mv mysql-5.1.40-linux-i686-icc-glibc23 /usr/local/mysql

```

1. 建立mysql用户

```
[root@localhost src]# useradd -s /sbin/nologin mysql

```

1. 初始化数据库

```
[root@localhost src]# cd /usr/local/mysql
[root@localhost mysql]# mkdir -p /data/mysql ; chown -R mysql:mysql /data/mysql
[root@localhost mysql]# ./scripts/mysql_install_db --user=mysql --datadir=/data/mysql

```

`--user` 定义数据库的所属主， `--datadir` 定义数据库安装到哪里，建议放到大空间的分区上，这个目录需要自行创建。这一步骤很关键，如果你看到两个 “OK” 说明执行正确，否则请仔细查看错误信息，如果你实在解决不了，请把问题发到论坛 ([http://www.aminglinux.com/bbs/forum-40-1.html](http://www.aminglinux.com/bbs/forum-40-1.html))阿铭会来帮你解决问题。

1. 拷贝配置文件

```
[root@localhost mysql]# cp support-files/my-large.cnf /etc/my.cnf

```

1. 拷贝启动脚本文件并修改其属性

```
[root@localhost mysql]# cp support-files/mysql.server  /etc/init.d/mysqld
[root@localhost mysql]# chmod 755 /etc/init.d/mysqld

```

1. 修改启动脚本

```
[root@localhost mysql]# vim /etc/init.d/mysqld

```

需要修改的地方有 “datadir=/data/mysql” （前面初始化数据库时定义的目录）

1. 把启动脚本加入系统服务项，并设定开机启动，启动mysql

```
[root@localhost mysql]# chkconfig --add mysqld
[root@localhost mysql]# chkconfig mysqld on
[root@localhost mysql]# service mysqld start

```

如果启动不了，请到 /data/mysql/ 下查看错误日志，这个日志通常是主机名.err. 检查mysql是否启动的命令为:

```
[root@localhost mysql]# ps aux |grep mysqld

```

## 安装php

这里要先声明一下，针对Nginx的php安装和针对apache的php安装是有区别的，因为Nginx中的php是以fastcgi的方式结合nginx的，可以理解为nginx代理了php的fastcgi，而apache是把php作为自己的模块来调用的。同样的，阿铭建议你使用5.3版本。php官方下载地址: [http://www.php.net/downloads.php](http://www.php.net/downloads.php)

1. 下载php

```
[rot@localhost httpd-2.2.24]# cd /usr/local/src
[root@localhost src]# wget http://am1.php.net/distributions/php-5.3.27.tar.gz

```

1. 解压php

```
[root@localhost src]# tar zxf php-5.3.27.tar.gz

```

1. 创建相关账户

```
[root@localhost src]# useradd -s /sbin/nologin php-fpm

```

1. 配置编译参数

```
[root@localhost src]# cd php-5.3.27
[root@localhost php-5.3.27]# ./configure \
--prefix=/usr/local/php \
--with-config-file-path=/usr/local/php/etc \
--enable-fpm \
--with-fpm-user=php-fpm \
--with-fpm-group=php-fpm \
--with-mysql=/usr/local/mysql \
--with-mysql-sock=/tmp/mysql.sock \
--with-libxml-dir \
--with-gd \
--with-jpeg-dir \
--with-png-dir \
--with-freetype-dir \
--with-iconv-dir \
--with-zlib-dir \
--with-mcrypt \
--enable-soap \
--enable-gd-native-ttf \
--enable-ftp \
--enable-mbstring \
--enable-exif \
--disable-ipv6 \
--with-pear \
--with-curl \
--with-openssl

```

该过程中，如果出现如下错误，请按照阿铭给出的解决办法解决，如果出现的错误阿铭并没有写出来，请参考上一章LAMP的php安装步骤([http://study.lishiming.net/chapter17.html#php](http://study.lishiming.net/chapter17.html#php))

错误信息:

```
configure: error: Please reinstall the libcurl distribution -
    easy.h should be in <curl-dir>/include/curl/

```

解决办法:

```
yum install -y libcurl-devel

```

1. 编译php

```
[root@localhost  php-5.3.27]# make

```

在这一步，你通常会遇到一些错误，没有关系，遇到错误是好事，这样可以增加你处理问题的经验。阿铭同样也遇到了错误:

```
/usr/bin/ld: cannot find -lltdl
collect2: ld returned 1 exit status
make: *** [sapi/fpm/php-fpm] 错误 1

```

阿铭是这样解决的:

```
yum install -y libtool-ltdl-devel

```

1. 安装php

```
[root@localhost  php-5.3.27]# make install

```

以上每一个步骤，如果没有完全执行正确，那么下一步是无法进行的，是否还记得判断执行是否正确的方法？ 使用 `echo $?` 看结果是否为 “0” , 如果不是，就是没有执行正确。

1. 修改配置文件

```
cp php.ini-production /usr/local/php/etc/php.ini
vim /usr/local/php/etc/php-fpm.conf

```

把如下内容写入该文件:

```shell
[global]
pid = /usr/local/php/var/run/php-fpm.pid
error_log = /usr/local/php/var/log/php-fpm.log
[www]
listen = /tmp/php-fcgi.sock
user = php-fpm
group = php-fpm
listen.owner = nobody      #此处必须是ngnix的用户，否则生产的sokets无法读取
listen.group = nobody      #此处必须是ngnix的组，否则生产的sokets无法读取
pm = dynamic
pm.max_children = 50
pm.start_servers = 20
pm.min_spare_servers = 5
pm.max_spare_servers = 35
pm.max_requests = 500
rlimit_files = 1024
```

注：此处可以用tpc/ip形式，listen改为`127.0.0.1:9000`，则在nginx.conf中也需改为这个

保存配置文件后，检验配置是否正确的方法为:

```
/usr/local/php/sbin/php-fpm -t

```

如果出现诸如 “test is successful” 字样，说明配置没有问题。

1. 启动php-fpm

```
cp /usr/local/src/php-5.3.27/sapi/fpm/init.d.php-fpm /etc/init.d/php-fpm
chmod 755 /etc/init.d/php-fpm
service php-fpm start
```

如果想让它开机启动，执行:

```
chkconfig php-fpm on

```

检测是否启动:

```
ps aux |grep php-fpm

```

看看是不是有很多个进程（大概20多个）。

## 安装nginx

(近期nginx报出一个安全漏洞，影响版本很广 CVE-2013-4547，所以之前的老版本都需要升级一下, 1.4.4, 1.5.7以及往后版本没有问题)

1. 下载nginx

```
cd /usr/local/src/
wget http://nginx.org/download/nginx-1.4.4.tar.gz

```

1. 解压nginx

```
tar zxvf nginx-1.4.4.tar.gz

```

1. 配置编译参数

```
cd nginx-1.4.4
./configure \
--prefix=/usr/local/nginx \
--with-http_realip_module \
--with-http_sub_module \
--with-http_gzip_static_module \
--with-http_stub_status_module  \
--with-pcre

```

1. 编译nginx

```
make

```

1. 安装nginx

```
make install
```

1. 编写nginx启动脚本，并加入系统服务

```
vim /etc/init.d/nginx
```

写入如下内容:

```
#!/bin/bash
# chkconfig: - 30 21
# description: http service.
# Source Function Library
. /etc/init.d/functions
# Nginx Settings

NGINX_SBIN="/usr/local/nginx/sbin/nginx"
NGINX_CONF="/usr/local/nginx/conf/nginx.conf"
NGINX_PID="/usr/local/nginx/logs/nginx.pid"
RETVAL=0
prog="Nginx"

start() {
        echo -n $"Starting $prog: "
        mkdir -p /dev/shm/nginx_temp
        daemon $NGINX_SBIN -c $NGINX_CONF
        RETVAL=$?
        echo
        return $RETVAL
}

stop() {
        echo -n $"Stopping $prog: "
        killproc -p $NGINX_PID $NGINX_SBIN -TERM
        rm -rf /dev/shm/nginx_temp
        RETVAL=$?
        echo
        return $RETVAL
}

reload(){
        echo -n $"Reloading $prog: "
        killproc -p $NGINX_PID $NGINX_SBIN -HUP
        RETVAL=$?
        echo
        return $RETVAL
}

restart(){
        stop
        start
}

configtest(){
    $NGINX_SBIN -c $NGINX_CONF -t
    return 0
}

case "$1" in
  start)
        start
        ;;
  stop)
        stop
        ;;
  reload)
        reload
        ;;
  restart)
        restart
        ;;
  configtest)
        configtest
        ;;
  *)
        echo $"Usage: $0 {start|stop|reload|restart|configtest}"
        RETVAL=1
esac

exit $RETVAL
```

保存后，更改权限:

```
chmod 755 /etc/init.d/nginx
chkconfig --add nginx
```

如果想开机启动，请执行:

```
chkconfig nginx on
```

1. 更改nginx配置

首先把原来的配置文件清空:

```
> /usr/local/nginx/conf/nginx.conf

```

“>” 这个符号之前阿铭介绍过，为重定向的意思，单独用它，可以把一个文本文档快速清空。

```
vim /usr/local/nginx/conf/nginx.conf

```

写入如下内容:

```shell
user nobody nobody;
worker_processes 2;
error_log /usr/local/nginx/logs/nginx_error.log crit;
pid /usr/local/nginx/logs/nginx.pid;
worker_rlimit_nofile 51200;

events
{
    use epoll;
    worker_connections 6000;
}

http
{
    include mime.types;
    default_type application/octet-stream;
    server_names_hash_bucket_size 3526;
    server_names_hash_max_size 4096;
    log_format combined_realip '$remote_addr $http_x_forwarded_for [$time_local]'
    '$host "$request_uri" $status'
    '"$http_referer" "$http_user_agent"';
    sendfile on;
    tcp_nopush on;
    keepalive_timeout 30;
    client_header_timeout 3m;
    client_body_timeout 3m;
    send_timeout 3m;
    connection_pool_size 256;
    client_header_buffer_size 1k;
    large_client_header_buffers 8 4k;
    request_pool_size 4k;
    output_buffers 4 32k;
    postpone_output 1460;
    client_max_body_size 10m;
    client_body_buffer_size 256k;
    client_body_temp_path /usr/local/nginx/client_body_temp;
    proxy_temp_path /usr/local/nginx/proxy_temp;
    fastcgi_temp_path /usr/local/nginx/fastcgi_temp;
    fastcgi_intercept_errors on;
    tcp_nodelay on;
    gzip on;
    gzip_min_length 1k;
    gzip_buffers 4 8k;
    gzip_comp_level 5;
    gzip_http_version 1.1;
    gzip_types text/plain application/x-javascript text/css text/htm application/xml;

server
{
    listen 80;
    server_name localhost;
    index index.html index.htm index.php;
    root /usr/local/nginx/html;

    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_pass unix:/tmp/php-fcgi.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME /usr/local/nginx/html$fastcgi_script_name;
    }

}

}
```

**nginx.conf**中可用`include vhosts/*.conf`来创建一个虚拟主机配置文件，此处含义为

保存配置后，先检验一下配置文件是否有错误存在:

```
/usr/local/nginx/sbin/nginx  -t

```

如果显示内容如下，则配置正确，否则需要根据错误提示修改配置文件:

```
nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful

```

启动nginx:

```
service nginx start

```

如果不能启动，请查看 “/usr/local/nginx/logs/error.log” 文件，检查nginx是否启动:

```
ps aux |grep nginx

```

看是否有进程。

## 测试是否解析php文件

创建测试文件:

```
vim /usr/local/nginx/html/2.php

```

内容如下:

```
<?php
    echo "测试php是否解析";
?>

```

测试:

```
[root@localhost nginx]# curl localhost/2.php
测试php是否解析[root@localhost nginx]#
```