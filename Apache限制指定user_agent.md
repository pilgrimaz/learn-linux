#### Apache限制指定user_agent

有些_user_agent_不是我们想要的，可以通过rewrite功能针对_%{HTTP_USER_AGENT}_来rewrite到404页，从而达到限制某些_user_agent_的请求。

配置如下

```shell
<IfModule mod_rewrite.c>
	RewriteEngine on
	RewreteCond %{HTTP_USER_AGENT} ^.*Firefox/4.0 [NC,OR]
	RewriteCond %{HTTP_USER_AGENT} ^.*tomato Bot/1.0* [NC]
    RewriteCond %{REQUEST_URI} !^/404*
    RewriteCond .* /404/html
<IfModule>
```

请注意，你的404.html千万别再跳转到其他页面了，否则很有可能就会死循环了。

其实rewrite到404.html并不是很好的办法，而apache的rewrite功能有一项就是forbidden，那就是F

配置如下

```shell
<IfMoudle mod_rewrite.c>
	RewriteEngine on
	RewriteCond %{HTTP_USER_AGENT} ^*Firefox* [NC,OR]
	RewriteCond %{HTTP_USER_AGENT} ^*Tomato Bot/1.0* [NC]
	RewriteRule .* [F]
</IfModule>
```

