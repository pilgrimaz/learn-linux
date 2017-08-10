#### memcache和memcached在php中的应用

memcache在php中编译

```
#  wget http://www.lishiming.net/data/attachment/forum/memcache-2.2.3.tgz
# tar zxvf memcache-2.2.3.tgz
# cd memcache-2.2.3

# /usr/local/php/bin/phpize

# ./configure --with-php-config=/usr/local/php/bin/php-config
# make
# make install
# cp modules/memcache.so /usr/local/php/ext/   //把memcache.so 拷贝至php的extension_dir下，#查看php extension_dir的方法是  /usr/local/php/bin/php -i |grep extension_dir
修改扩展路径，在php.ini中修改：
extension_dir = "/usr/local/php/ext"
然后在php.ini 中添加
extension = memcache.so
```
保存后可以利用 /usr/local/php/bin/php -m  检测和查看具体的参数


memcached 的编译安装
wget http://syslab.comsenz.com/downloads/linux/memcached-1.4.5.tar.gz

tar zxvf memcached-1.2.8.tar.gz
cd  memcached-1.2.8
./configure --prefix=/usr/local/memcached
make && make install

启动：

/usr/local/memcached/bin/memcached -m 2048 -p 11211 -l 127.0.0.1 -d -u www
-m  后边指定memecached使用多少内存，单位是M
-p  指定memcached 启动端口
-l  指定绑定的IP
-u  指定以某个账户的身份启动

注意事项：

在安装memcached之前需要安装libevent支持：


```
#wget http://www.monkey.org/~provos/libevent-1.3b.tar.gz

#cd libevent-1.3b

#./configure --prefix=/usr/local/libevent

#make && make install
```


注意：可能在启动memcached的时候报以下错误：
```
#/usr/local/memcached/bin/memcached -m 2048 -p 11211 -l 127.0.0.1 -d -u www

/usr/local/memcached/bin/memcached: error while loading shared 
libraries: libevent-1.3b.so.1: cannot open shared object file: No such 
file or directory
```
那么需要把 libevent-1.2a.so.1 拷贝或链接到 /usr/lib 中，否则 memcached 无法正常加载。  cp /usr/local/libevent/lib/libevent-1.3b.so.1 /usr/lib



如果还是出现相同错误，则把/usr/lib放到动态库文件中：
```
#echo "/usr/lib" >> /etc/ld.so.conf

#ldconfig
```
至此，安装memcached结束！，再启动memcached就不报找不到libevent-1.3b.so模块了												