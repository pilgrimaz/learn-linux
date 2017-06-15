在原来的sever站点上面增加

```shell
 server {
 listen 80 default_server;
 server_name b.com;
 rewrite ^ http://www.a.com$request_uri?;      #跳转，当使用IP去访问的时候，跳转到域名访问，而不是IP
}
```



也可以用下面的

```SHELL
 server {
 listen 80 default_server;
 server_name b.com;
 return 403;      #用403或者444，视乎不太友好
 }
```