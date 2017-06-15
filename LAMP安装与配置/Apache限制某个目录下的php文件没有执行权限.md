#### apache限制某个目录下的php文件没有执行权限

为了安全起见，有时我们需要限制网站的某些目录对于php脚本不能执行

有两种方法可以参考：

1. 使用_.htaccess_文件限制

   在要限制php执行的目录下，创建_.htaccess_文件，加入内容

   `php_flag engine off`

2. 使用apache的配置文件_http.conf_

   在相关的虚拟主机段，加入，

   ```shell
   <Directory /www/htdocs/path>
   	php_admin_flag engine off
   	<filematch "(.*)php">
   		Order deny,allow
   		Deny from all
   		Allow from 127.0.0.1
   	</filematch>
   <Directory>
   ```

   ​