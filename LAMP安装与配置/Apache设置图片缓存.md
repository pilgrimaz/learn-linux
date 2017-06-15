#### Apache设置图片缓存

在http.conf中加入

```shell
<IfMoudle mod_expires.c>
	ExpiresActive on
	ExpiresByType image/gif "access plus 1 days"
	ExpiresByType image/jpeg "access plus 24 hours"
	ExpiresByType image/png "access plus 24 hours"
	ExpiresByType text/css "now plus 2 hour"
	ExpiresByType application/x-javascript"now plus 2 hours"
	ExpiresByType application/x-shockwace-flash"now plus 2 hours"
	ExpiresDefault "now plus 0 min"
</IfModule>
```

注释：

Expires 语法如下：

ExpiresByType type/encoding "<base>[plus]<num><type>}*"

其中<base>是下列之一:
access
now(等价于'access')
modification

plus关键字是可选的。

<num>必须是整数












