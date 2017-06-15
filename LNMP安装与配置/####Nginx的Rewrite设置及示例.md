####Nginx的Rewrite设置及示例
Nginx以其良好的并发性能，目前正在逐渐取代Apache成为大家的Web server首选，但是Nginx目前的中文资料很少，需要大家努力贡献。
下面我介绍一下Nginx的Rewrite模块设置及Wordpress和Discuz的示例。Nginx的Rewrite规则比Apache的简单灵活多了，从下面介绍可见一斑。
首先，Nginx可以用if进行条件匹配，语法规则类似C，举例如下：
`if ($http_user_agent ~ MSIE) {rewrite  ^(.*)$  /msie/$1  break;}`
1. 正则表达式匹配，其中：
   ~  为区分大小写匹配 
   ~* 为不区分大小写匹配 
   !~和!~*分别为区分大小写不匹配及不区分大小写不匹配
2. 文件及目录匹配，其中：
   -f和!-f用来判断是否存在文件 
   -d和!-d用来判断是否存在目录 
   -e和!-e用来判断是否存在文件或目录 
   -x和!-x用来判断文件是否可执行
   如：
   `if (!-f $request_filename{proxy_pass http://127.0.0.1;}`
   其次，Nginx的Rewrite规则与Apache几乎完全一致，所不同的是最后的flag标记，举例如下：
   `rewrite ^/feed/$ http://feed.shunz.net last;`
   flag标记有：
   last 相当于Apache里的[L]标记，表示完成rewrite，不再匹配后面的规则 
   break 与last类似 
   redirect 返回302临时重定向 
   permanent 返回301永久重定向
   Wordpress的重定向规则：
   ```shell
   if (!-e $request_filename) {rewrite ^/(index|atom|rsd)\.xml$ http://feed.shunz.net
    last;rewrite ^([_0-9a-zA-Z-]+)?(/wp-.*) $2 last;rewrite 
   ^([_0-9a-zA-Z-]+)?(/.*\.php)$ $2 last;rewrite ^ /index.php last;}  
   ```