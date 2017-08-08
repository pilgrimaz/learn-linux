要支持 php就要安装phpredis 下载地址**https://pecl.php.net/package/redis**

**phpredis 安装步骤 **

**1、解压安装并进入Redis目录**

[root@Redis ~]# tar xzf redis-2.2.8.tgz
[root@Redis ~]# cd redis-2.2.8

**2、在Redis文件夹下，生成configure配置文件**

[root@Redis redis-2.2.8]# /usr/local/php/bin/phpize
Configuring for:
PHP Api Version: 20090626
Zend Module Api No: 20090626
Zend Extension Api No: 220090626
记得sudo 切换root用户
[root@Redis redis-2.2.8]# ./configure --with-php-config=/usr/local/php/bin/php-config
[root@Redis redis-2.2.8]# make
[root@Redis redis-2.2.8]# make install

**3、在PHP配置文件php.ini里面加载Redis扩展**

extension=redis.so

重启php-fpm

**4****、查看phpinfo，Redis扩展是否加载**

```php
<?php
phpinfo();
?>
```



　查看版本里面有没有显示redis的版本

 

　　

**redis安装步骤**

下载安装包目录** https://redis.io/download**

**1.解压编译**

1、tar -xf redis-stable.tar.gz 

cd redis-stable

make

 yum install gcc-c++ tcl

make test

2、安装常遇问题

前面3步应该没有问题，主要的问题是执行make的时候，出现了异常。

异常一：

make[2]: cc: Command not found

异常原因：没有安装gcc

解决方案：yum install gcc-c++

 

异常二：

zmalloc.h:51:31: error: jemalloc/jemalloc.h: No such file or directory

异常原因：一些编译依赖或原来编译遗留出现的问题

解决方案：make distclean。清理一下，然后再make。

 

在make成功以后，需要make test。在make test出现异常。

异常一：

couldn't execute "tclsh8.5": no such file or directory

异常原因：没有安装tcl

解决方案：yum install -y tcl。

 

 

在make成功以后，会在src目录下多出一些可执行文件：redis-server，redis-cli等等。

方便期间用cp命令复制到usr目录下运行。

cp redis-server /usr/local/bin/

cp redis-cli /usr/local/bin/

然后新建目录，存放配置文件

mkdir /etc/redis

mkdir /var/redis

mkdir /var/redis/log

mkdir /var/redis/run

mkdir /var/redis/6379

 

在redis解压根目录中找到配置文件模板，复制到如下位置。

cp redis.conf /etc/redis/6379.conf

通过vi命令修改

daemonize yes

pidfile /var/redis/run/redis_6379.pid

logfile /var/redis/log/redis_6379.log

dir /var/redis/6379

最后运行redis：

/usr/local/bin/redis-server /etc/redis/6379.conf

 

##  使用Redis启动脚本设置开机自启动

### 启动脚本

推荐在生产环境中使用启动脚本方式启动redis服务。启动脚本 `redis_init_script` 位于位于Redis的 `/utils/` 目录下。

```
#大致浏览下该启动脚本，发现redis习惯性用监听的端口名作为配置文件等命名，我们后面也遵循这个约定。
#redis服务器监听的端口
REDISPORT=6379
#服务端所处位置，在make install后默认存放与`/usr/local/bin/redis-server`，如果未make install则需要修改该路径，下同。
EXEC=/usr/local/bin/redis-server
#客户端位置
CLIEXEC=/usr/local/bin/redis-cli
#Redis的PID文件位置 修改部分与前面对应
PIDFILE=/var/redis/run/redis_${REDISPORT}.pid
#配置文件位置，需要修改
CONF="/etc/redis/${REDISPORT}.conf"
```

### 配置环境

 1. 将启动脚本复制到/etc/init.d目录下，本例将启动脚本命名为redisd（通常都以d结尾表示是后台自启动服务）。

```
cp redis_init_script /etc/init.d/redisd
```

 2.  设置为开机自启动

**此处直接配置开启自启动 chkconfig redisd on 将报错误： service redisd does not support chkconfig **

```
#!/bin/sh
# chkconfig:   2345 90 10  # 加上这句话，且注释掉
# description:  Redis is a persistent key-value database
#
```

 再设置即可成功。

```
#设置为开机自启动服务器
chkconfig redisd on
#打开服务
service redisd start
#关闭服务
service redisd stop


```

 测试phpredis 是否好用

```php
?php
$redis = new redis();
$redis->connect('127.0.0.1', 6379);
$redis->auth(123456);
$redis->set('foo','bar');
echo $redis->get('foo');
```



>详细参考 http://www.cnblogs.com/richerdyoung/p/6701150.html