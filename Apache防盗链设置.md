#### Apache防盗链设置

位置一般情况下在`/usr/local/apache/conf/httpd.cpnf`或者apache2.2的在`/usr/local/apache2/conf/extra/httpd-vhost.conf`

添加

```shell
SetEnvIfNoCase Refer "^http://www.ccvita.com" local_ref
SetnvIfNoCase Refer "^http://ccvita.com" local_ref
<filesmatch "\.(txt|doc|mp3|zip|rar|jpg|gif)>
Order Allow,Deny
Allow from env=local_ref
</filesmatch>
```



----

还有一种写法，使用正则的，这种写法在各个版本的apache比较通用。

写法是



```shell
SetEnvIfNoCase Refer "^http://.*\.yourdomin\.com" local_ref
SetnvIfNoCase Refer ".*\.yourdomin\.com" local_ref
<filesmatch "\.(txt|doc|mp3|zip|rar|jpg|gif)>
Order Allow,Deny
Allow from env=local_ref
</filesmatch>
```

