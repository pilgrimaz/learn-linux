#### 针对访问uri限制ip

在虚拟主机配置文件中加入如下字段：

```shell
<filesmatch "(.*)admin(.*)">
			Order deny,allow
			Deny from all
			Allow from 127.0.0.1
			Allow from 2.2.2.2
</filematch>
```

假如该虚拟机的域名为domain.com，这样配置后，除了127.0.0.1和2.2.2.2外，其他IP访问以下类似的uri时都会直接禁止的。
http://domain.com/1212admin.txt
http://domain.com/admin.php
http://domain.com/1212/admin.html
等