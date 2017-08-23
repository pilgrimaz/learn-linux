Memcached介绍
- 是国外社区网站LiveJounal团队开发，通过缓存数据库查询结果，减少数据库访问次数，从而提高动态web站点性能。
- 基于C/S架构，协议简单
- 基于libevent的事件处理
- 自主内存存储处理(slab allowcation)
- 数据过期方式：Lazy Expiration 和 LRU

Memcached安装
yum install -y epel-release
启动 /etc/init.d/memcached start
ps aux|grep memcached
memcached -d -p 11211 -u memcached -m 64  -c 1024 -P /var/run/memcached/memcached.pid
相关参数在/etc/init.d/memcached和/etc/sysconfig/memcached中定义


memcached启动参数
-d 启动一个守护进程(后台运行）
-m 是分配给memcache使用的内存数量，单位是MB
-u 是运行memcache的用户，如果当前为root的话，需要使用此参数指定用户
-l 是监听的服务器IP地址
-p 是设置memcache监听的端口，默认是11211
-c 选项是最大运行的并发连接数，默认是1024
-P 是设置保存memcache的pid文件

查看memcached运行状态
memcached-tool 127.0.0.1:11211 stats
或者 echo stats |nc 127.0.0.1 11211 需要安装nc工具 yum install -y nc
若安装libmemecached后，可以使用命令
memstat --servers=127.0.0.1:11211 查看memcached服务状态

memcached命令行
```mariadb
telnet 127.0.0.1 11211
set key2 0 60 3
abc
STORED
get key2
VALUE key2 0 3
abc
END
```

memcached语法规则
<command name><key><flags><exptime><bytes>\r\n<datablock>\r\n
注：在windows下是ENTER键
a) <command name>可以是"set","add","replace"。
"set"表示按照相应的<key>存储该数据，没有的时候增加，有的时候覆盖。
"add"表示按照相应的<key>添加该数据，但是如果该<key>已经存在则会操作失败。
"replace"表示按照相应的<key>替换数据，但是如果该<key>不存在则操作失败。
b)<key>客户端需要保存数据的key。
c)<flags>是一个16位的无符号的整数(以十进制的方式表示)。
 该标志将和需要存储的数据一起存储，并在客户端get数据时返回。
 客户可以将此标志用做特殊用途，此标志对服务器来说是不透明的。
d)<exptime>过期的时间。
 若为0表示存储的数据永远不过期(但可被服务器算法：LRU等替换)。
 如果非0(unix时间或者距离此时的秒数)，当过期后，服务器可以保证用户得不到该数据(以服务器时间为标准)。
e)<bytes>需要存储的字节数，当用户希望储存空数据时<bytes>可以为0
f)<data block>需要存储的内容，输入完成后，最后客户端需要加上“\r\n"(直接点击Enter)作为"命令头"的结束标志。

PHP连接memcached
先安装php的memcache扩展

memcached 实现session共享
- 本实例是在lamp/lnmp环境下实现
- 编辑php.ini添加两行
 session.save_handler = memcache
 session.save_path = "tcp://192.168.0.9:11211"  (memcache路径) 
- 或者httpd.conf中对应的虚拟主机中添加
 php_value session.save_handler "memcache"
 php_value session.save_path "tcp://192.168.0.9:11211"
- 或者php-fpm.conf对应的pool中添加
 php_value[session.save_handler] = memcache
 php_value[session.save_path] = "tcp://192.168.0.9:11211"