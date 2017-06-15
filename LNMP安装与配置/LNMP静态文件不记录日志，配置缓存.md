#### LNMP静态文件不记录日志，配置缓存
```shell
location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$

	{
      expires 30d;
      access_log off;
	}
	
location ~ .*\.(js|css)?$
	{
      expires 12h;
      access_log off;
	}
```