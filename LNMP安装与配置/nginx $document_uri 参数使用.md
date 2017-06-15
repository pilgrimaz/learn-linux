#### nginx $document_uri 参数使用

$document_uri  表示访问的url 

现在我的需求是，访问 www.abc.com  请求到 www.abc.com/abc/

在nginx配置文件中加入

```shell
 if ($document_uri !~ 'abc')
    {
            rewrite ^/(.*)$ http://www.abc.com/abc/$1 permanent;
    }
```
而不是单独加一句  rewrite ^/(.*)$ http://www.abc.com/abc/$1 permanent;
如果只加rewrite 规则，而不限定条件，那么会造成死循环。  会访问到   http://www.abc.com/abc/abc/abc/abc/.... 