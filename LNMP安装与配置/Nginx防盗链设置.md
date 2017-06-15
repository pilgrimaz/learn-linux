#### Nginx防盗链设置

在nginx.conf中的server部分中添加如下代码

```shell
location ~* ^.+\.(gif|jpg|png|swf|flv|rar|zip|doc|pdf|gz|bz2|jpeg|bmp|xls)$ {   
                valid_referers none blocked server_names  *.taobao.com *.baidu.com *.google.com *.google.cn *.soso.com ;  // 对这些域名的网站不进行盗链。
                if ($invalid_referer) {
#                        rewrite   ^/   http://www.52blackberry.com/403.html;
#                        return 403;
                        rewrite ^/ http://www.example.com/nophoto.gif;
                        }
                }
```

其中 rewrite ^/ 后边可以是一个错误页面，如上边那一行，也可以是一个图片，如下面那个。

对于开头location 部分有的是这样的形式  location ~ .*\.(gif|jpg|png)，经我验证都可以实现。