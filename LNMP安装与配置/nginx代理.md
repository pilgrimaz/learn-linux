####nginx代理
```
upstream bbs.aaa.cn{
            server 1.2.3.1:80;
            server  1.2.3.4:80;
        }


        server {
            listen 80;
            server_name bbs.aaa.cn;

            location / {
                proxy_pass      http://bbs.aaa.cn/;         #核心代码
                proxy_set_header Host   $host;
                proxy_set_header X-Real-IP      $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            }
#            access_log  /home/logs/bbs.access combined;
        }


        upstream blog.aaa.cn{
            server  1.2.3.1:80;
            server  1.2.3.4:80;
        }

        server {
            listen 80;
            server_name blog.aaa.cn;

            location / {
                proxy_pass      http://blog.aaa.cn/;
                proxy_set_header Host   $host;
                proxy_set_header X-Real-IP      $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            }
#            access_log  /home/logs/ss.access combined;
        } 
```
经测试发现，可以用nginx代理一个服务器上所有域名，方法如下：
1. 主配置文件不需要更改任何配置
2. 在vhosts目录下需要建立两个文件，一个是servername 列表文件，一个是虚拟主机配置文件
   两个文件内容分别为
   （1） servername
   server_name www.123.net.cn  www.alsdjfl.com   www.asdfa1.com;  //就这么简单一行，当然这个server_name 还可以继续添加的

（2） 虚拟主机配置文件
```
server {
            listen 80;
            include vhosts/servername; // 这里的文件就是上边那个servername列表文件
            location / {
                proxy_pass     http://1.2.1.2/;  //这里就是需要做代理的服务器ip地址了
                proxy_set_header Host   $host;
                proxy_set_header X-Real-IP      $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            }
            access_log  /dev/null;
        } 
```



针对apache和nginx共同协作的网站架构，也可以根据动态内容和静态内容的请求来做负载均衡
如：
        upstream apache.com{
            server  1.2.3.1:80;
            server  1.2.3.4:80;
        }
location   ~*   .*\.(php|phps|jsp)$
        {
                proxy_pass  http://apache.com;
        } 


​        
跳转到163案例(核心也是 proxy_pass)
```
   ## Basic reverse proxy server ##  
    ## backend for 16.32  ##  
   ##跳转到163测试
    upstream uicps  {  
    #    server 192.168.16.32:59002 weight=1;  
         server www.163.com;  
    }  
      
    ## Start 16.32 ##  
    server {  
        listen 9000;  
        server_name  localhost;  
      
    #    access_log  logs/proxy34.access.log  main;  
    #    error_log  logs/proxy34.error.log;  
        root   html;  
        index  index.html index.htm index.php;  
      
        ## send request back to 16.32 ##  
        location / {  
            proxy_pass  http://uicps;  
      
            #Proxy Settings  
            proxy_redirect     off;  
            proxy_set_header   Host www.163.com;     
            proxy_set_header   X-Real-IP        $remote_addr;  
            proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;  
            proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;  
            proxy_max_temp_file_size 0;  
            proxy_connect_timeout      90;  
            proxy_send_timeout         90;  
            proxy_read_timeout         90;  
            proxy_buffer_size          4k;  
            proxy_buffers              4 32k;  
            proxy_busy_buffers_size    64k;  
            proxy_temp_file_write_size 64k;  
       }  
    }  
```