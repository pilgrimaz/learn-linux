#### Nginx实现域名跳转

有时候有这样的需求，当两个或多个域名都可以访问同一个站点，但是我们想当访问B域名时，使其自动跳转至A域名。其实nginx很容易实现这样的功能：

（以下资料来源于互联网）

nginx中进行301重定向(301 redirect)是非常容易的。比方说要将www.caipanzi.com永久性重定向至caipanzi.com，有两种方法



1.方法A

```shell
server {
​    server_name caipanzi.com www.caipanzi.com;
​    if ($host != 'caipanzi.com' ) {
​        rewrite  ^/(.*)$  http://caipanzi.com/$1  permanent;
​        }
}
```
2.方法B(为带www的域名单独设一条server规则)
```shell
server {
​    server_name  www.caipanzi.com;
​    rewrite ^(.*) http://caipanzi.com$1 permanent;
}								
```
