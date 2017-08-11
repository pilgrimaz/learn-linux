#### LAMP中php相关配置

1. 配置_disable_functions_ 禁用函数列表

   位于php配置文件_php.ini_中

   ```shell
   disable_functions=eval,assert,popen,passthru,escapeshellarg,escapeshellcmd,passthru,exec,system,chroot,scandir,chgrp,chown,shell_exec,proc_get_status,ini_alter,ini_restore,dl,pfsockopen,openlog,syslog,readlink,symlink,leak,popepassthru,stream_sockt_server,popen,proc_open,proc_close,proc_get_status,fsocket
   ```

2. 配置error_log错误日志级别,同样在*php.ini*

   ```shell
   display_error=off
   log_errors=on
   error_log=/path/to/logfile
   error_reporting = E_ALL & ~E_NOTICE
   ```

   php错误日志级别参考：

   ; E_ALL             所有错误和警告（除E_STRICT外）

   ; E_ERROR           致命的错误。脚本的执行被暂停。

   ; E_RECOVERABLE_ERROR    大多数的致命错误。

   ; E_WARNING         非致命的运行时错误，只是警告，脚本的执行不会停止。

   ; E_PARSE            编译时解析错误，解析错误应该只由分析器生成。

   ; E_NOTICE          脚本运行时产生的提醒（往往是我们写的脚本里面的一些bug，比如某个变量没有定义），这个错误不会导致任务中断。

   ; E_STRICT          脚本运行时产生的提醒信息，会包含一些php抛出的让我们要如何修改的建议信息。

   ; E_CORE_ERROR      在php启动后发生的致命性错误

   ; E_CORE_WARNING    在php启动后发生的非致命性错误，也就是警告信息

   ; E_COMPILE_ERROR    php编译时产生的致命性错误

   ; E_COMPILE_WARNING  php编译时产生的警告信息

   ; E_USER_ERROR       用户生成的错误

   ; E_USER_WARNING    用户生成的警告

   ; E_USER_NOTICE      用户生成的提醒

   ​

   & 表示并且

   ~ 表示非

   | 表示或者


   比如： `error_reporting  =  E_ALL & ~E_NOTICE`  表示错误级别为E_ALL 并且除了E_NOTICE 

3. 配置open_basedir(目录隔离，只能访问指定目录中的文件)

   `php.ini:open_basedir = /dir1/:/dir2/`

   `httpd.conf:php_admin_value open_basedir "/dir1/:dir2/"`

   注：多站点相互隔离可到虚拟主机配置文件httpd_hosts中去添加

4. 安装php的扩展模块

   memcache在php中编译

   ```shell
   wget 安装包
   tar zxvf                       解压安装包
   cd memcache-2.2.3.tgz
   /usr/local/php/bin/phpize
   ./configure --with-php-config=/usr/local/php/bin/php-config
   make
   make install
   cp modules/memcache.so /usr/local/php/ext/
   ```

   然后在php.ini中添加

   ectension = memcache.so

   修改扩展路径为：extension_dir="/usr/local/php/ext"

   保存后可以利用 /usr/local/php/bin/php-cgi -m 检测和查看具体的参数

5. *apache2/conf/extra/httpd-mpm.conf*介绍

   ```shell
   <IfModule mpm_prefork_module>
       StartServers          5     开始进程数
       MinSpareServers       5     最小空闲进程
       MaxSpareServers      10     最大空闲进程
       MaxClients          150     最大子进程
       MaxRequestsPerChild   0     最大访问次数后关闭该进程
   </IfModule>
   ```