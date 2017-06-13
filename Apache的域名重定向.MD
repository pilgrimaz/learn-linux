#### Apache的域名重定向

需求：要把访问域名www.domain1.com的域名转发到www.dmian2.com上

实现：

在相关的==虚拟主机==中增加

```shell
<IfMoudule mod_rewrite.c>
	RewriteEngine on
	RewriteCond %{HTTP_HOST} ^www.domain1.com$
	RewriteRule ^(.*)$ http://www.domain2.com/$1[R=301，L]
<Ifmodule>
```

如果是多个域名重定向到一个域名

```shell
<IfMoudule mod_rewrite.c>
	RewriteEngine on
	RewriteCond %{HTTP_HOST} ^www.domain1.com$[OR]
	RewriteCond %{HTTP_HOST} ^www.domain2.com$
	RewriteRule ^(.*)$ http://www.domain2.com/$1[R=301，L]
<Ifmodule>
```

)