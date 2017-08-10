#### PHP添加编译时忘添加的模块方式

```
比如添加bcmath模块：(在编译时没有添加 --enable-bcmath)

# cd php-5.3.10/ext/bcmath
# /usr/local/php/bin/phpize
# ./configure --with-php-config=/usr/local/php/bin/php-config
# make && make install
# cp /usr/local/php/lib/php/extensions/no-debug-non-zts-20060613/bcmath.so
/usr/lib/php/modules
# vi /usr/local/php/etc/php.ini
[bcmath]
extension=bcmath.so
:wq

重启apache：

# /usr/local/apache/bin/apachectl restart
```