#### 配置apache的访问日志

在虚拟主机配置文件`httpd-vhsot`中虚拟主机配置中加入：

```shell
Errorlog "|/usr/local/apache/bin/rotatelogs -l /usr/local/apache/logs/oem.discuz.qq.com-error_%Y%m%d.log 86400"
	SetEnvIf Request_URI ".*\.gif$" image-request
	SetEnvIf Request_URI ".*\.jpg$" image-request
	SetEnvIf Request_URI ".*\.png$" image-request
	SetEnvIf Request_URI ".*\.bmp$" image-request
	SetEnvIf Request_URI ".*\.swf$" image-request
	SetEnvIf Request_URI ".*\.js$" image-request
	SetEnvIf Request_URI ".*\.css$" image-request
	CustomLog "|/usr/local/apache/bin/rotatelogs -l /usr/local/apache/logs/ome.discuz.qq.com-access_%Y%m%d.log 86400" combine env=!image-request
```

