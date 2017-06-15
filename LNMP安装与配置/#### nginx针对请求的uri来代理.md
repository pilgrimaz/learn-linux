#### nginx针对请求的uri来代理

场景：1台nginx去代理4台apache

需求：根据不同的请求uri 代理到不同的apache



nginx的配置文件为

```shell
 upstream aa.com {         
                      server 192.168.0.121;
                      server 192.168.0.122;  
     }
    upstream bb.com {  
                       server 192.168.0.123;
                       server 192.168.0.124;
        }
    server {
        listen       80;
        server_name  www.abc.com;
        location ~ aa.php
        {
            proxy_pass http://aa.com/;
            proxy_set_header Host   $host;
            proxy_set_header X-Real-IP      $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
         location ~ bb.php
        {
              proxy_pass http://bb.com/;
              proxy_set_header Host   $host;
              proxy_set_header X-Real-IP      $remote_addr;
              proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
          }
}
```
