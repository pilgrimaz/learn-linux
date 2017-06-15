####nginx 使用 user_agent 控制客户端访问
```shell
location / {
            if ($http_user_agent ~ 'MSIE 6.0'){
                return 403;
               }
             }
```
今天遇到攻击，user_agent 为 IE 5.0 于是，用了这个方法。一开始说啥也不生效。后来终于搞定了。
我只是把以下几行放到了，最上面。  
```shell
if ($http_user_agent ~ 'MSIE 5.0'){
               return 403;
        }
```
**可以放到server_name 下												