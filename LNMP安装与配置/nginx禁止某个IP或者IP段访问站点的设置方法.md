#### nginx禁止某个IP或者IP段访问站点的设置方法

首先建立下面的配置文件放在nginx的conf目录下面,命名为deny.ip
```shell
cat  deny.ip
deny 192.168.1.11;
deny 192.168.1.123;
deny 10.0.1.0/24;
```
在nginx的配置文件nginx.conf中加入：include deny.ip; 
重启一下nginx的服务：/usr/local/nginx/sbin/nginx  reload 就可以生效了。 
deny.ip 的格式中也可以用deny all; 
如果你想实现这样的应用，除了几个IP外，其他全部拒绝，
那需要你在deny.ip 中这样写
allow 1.1.1.1; 
allow 1.1.1.2;
deny all;								