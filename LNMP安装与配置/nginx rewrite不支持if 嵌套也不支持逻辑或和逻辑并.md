#### nginx rewrite不支持if 嵌套也不支持逻辑或和逻辑并

如题，apache的rewrite是支持或者的，用个OR就可以，如果不加OR，多个RewriteCond 罗列累加就是并且的意思。然后nginx的rewrite就没有这么好了。那么如何去实现这样复杂的功能呢？这就用到了标记功能。



现在出一个简单的需求： 要求访问uri以 /abc/开头的请求，并且user_agent带有ie6或者firefox关键词的请求需要禁止访问。

实现方法为：

```shell
set $rule 0;
                if ($document_uri ~ '^/abc')
                {
                        set $rule "${rule}1";
                }
                if ($http_user_agent ~* 'ie6|firefox')
                {
                       set $rule "${rule}2";
                }
                if ($rule = "012")
                {
                        deny all;
                }
```
这样就可以实现了。
然后在我实践过程中，发现一个问题，就是如果定义超过3条rule，当条件中包含两条和两条以上的规则同时存在是，需要把两条规则的条件写到第4条规则前面。
例如，有一个这样的需求：实现rewrite的总前提是，所有请求必须以^/abc 目录为开头。其余规则如下：
1. user_agent 包含 'ipone' 或者'ipad'  或者'ipod' 的请求需要把 *htm 转发为 *html;
2. user_agent 不包含 'ipone' 或者'ipad'  或者'ipod' 并且user_agent 包含'ucweb'的请求需要把*htm 转发为 *html;
3. user_agent 不包含 'ipone' 或者'ipad'  或者'ipod' 并且user_agent 不包含'ucweb'的请求需要把*html 转发为 *htm;

规则语句为：
```shell
set $rule 2;
        if ($document_uri ~* '^/abc')
        {
            set $rule "${rule}1";
        }
        if ($http_user_agent ~* 'ipad|iphone|ipod')
        {
            set $rule "${rule}2";
        }
        if ($rule = "212")
        {
            rewrite ^(.*)\.htm$ $1\.html redirect;
        }
        if ($http_user_agent !~* 'ipad|iphone|ipod')
        {
            set $rule "${rule}3";
        }
        if ($http_user_agent !~* 'ucweb')
        {
            set $rule "${rule}4";
        }
        if ($http_user_agent ~* 'ucweb')
        {
            set $rule "${rule}5";
        }
        if ($rule = "2134")
        {
            rewrite ^(.*)\.html$ $1\.htm redirect;
        }
        if ($rule = "2135")
        {
            rewrite ^(.*)\.htm$ $1\.html redirect;
        }
```
注意，上面规则把 $rule = "212" 放到了上面，如果放到下面则不能实现跳转。
