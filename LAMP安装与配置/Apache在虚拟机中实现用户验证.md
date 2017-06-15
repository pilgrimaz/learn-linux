#### apache在虚拟机中实现用户验证

**方式一：**

在虚拟主机配置文件`httpd-vhost`中，需要加入：

```shell
<Directory /data/web/test>
	AllowOverride AuthConfig
</Directory>
```

然后在虚拟主机的主目录，`DocumentRoot`目录下

```shell
vi /data/web/test/.htaccess
```

加入

```shell
AuthName "frank share web"
AuthType Basic
AuthUserFile /data/web/test/htpasswd
require valid-user
```

保存后，然后

创建Apache的验证用户

```shell
htpasswd -c /data/web/test/.htpasswd test
```

\#第一次创建用户要用到`-c`参数  第二次添加用户，就不用`-c`参数

如果你想修改密码，可以如下

```shell
htpasswd -m .htppasswd test2
```

重启Apache，即可。

到此，你已经配置完成，下嘛介绍另一种方式

*******

```shell
vi http.conf
```

在相应的虚拟主机配置文件段，加入：

```shell
<Directory *>
		AllowOverride AuthConfig
		AuthName "自定义的"
		AuthType Basic
		AuthUserFile /data/.htpasswd //这个目录可以随便写一个，没有限制
	require valid-user
</Directory>
```
[^zhushi]
保存后，然后创建apache的验证用户

```shell
htpasswd -c /data/.htpasswd test
```

注：匹配文件用filesmatch





smqingk


