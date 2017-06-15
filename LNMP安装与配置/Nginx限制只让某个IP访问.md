#### Nginx限制只让某个IP访问
```shell
server 
{
​    listen       80;
​    server_name  www.aldjflas.cn;
​    access_log   /home/logs/bbs/access.log combined buffer=32k;
​    error_log    /home/logs/bbs/error.log warn;
​    index           index.html index.htm index.php;
​    root            /data/www/wwwroot/bbs;
​    allow          219.232.244.234;
​    deny           all;
 }								
```