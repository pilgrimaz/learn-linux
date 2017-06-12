1. 访问的文档权限不够。要755以上权限。解决方法：用命令chmod 755 /var/www/ 或其他相应目录。
2. SELinux或防火墙的原因。解决方法：先关闭SELinux和让防火墙通过WWW服务。
3. 虚拟主机配置错误。例如我遇到过一次的：
httpd.conf里加载了虚拟主机的配置文件：
```
# Virtual hosts
Include conf/extra/httpd-vhosts.conf
```
而`conf/extra/httpd-vhosts.conf`并没有配置好，而且虚拟主机功能暂时还没有用，所以把`Include conf/extra/httpd-vhosts.conf`注释掉，重启apache后正常了。
解决方法：重新配置虚拟主机或暂时关闭。

4. DocumentRoot的设置。解决方法如下：
打开 apache的配置文件httpd.conf，找到这段代码：
```css
<Directory />
Options FollowSymLinks
AllowOverride None
Order deny,allow
Deny from all
</Directory>
```
有时候由于配置了php后，这里的`“Deny from all”`已经拒绝了一切连接。把该行改成`“allow from all”`，修改后的代码如下，问题解决。
```
<Directory />
Options FollowSymLinks
AllowOverride None
Order deny,allow
Allow from all
</Directory>
```