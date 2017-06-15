#### nginx代理--根据访问的目录来区分后端的web

我的需求： 当请求的目录是 /aaa/ 则把请求发送到机器a，当请求的目录为/bbb/则把请求发送到机器b，除了目录/aaa/与目录/bbb/外，其他的请求发送到机器b



我的配置文件内容为：

```shell
    upstream aaa.com
    {
                server 192.168.111.6;
    }
    upstream bbb.com
    {
                server 192.168.111.20;
    }
    server {
            listen 80;
            server_name li.com;
            location /aaa/
            {
                proxy_pass http://aaa.com/aaa/;
                proxy_set_header Host   $host;
                proxy_set_header X-Real-IP      $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            }
            location /bbb/
            {
                proxy_pass http://bbb.com/bbb/;
                proxy_set_header Host   $host;
                proxy_set_header X-Real-IP      $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            }
            location /
            {
                proxy_pass http://bbb.com/;
                proxy_set_header Host   $host;
                proxy_set_header X-Real-IP      $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            }
    }
```
说明：
1. 以上配置文件中的 aaa.com 以及 bbb.com 都是自定义的，随便写。
2. upstream 中的server 可以写多个，例如
```shell
upstream aaa.com
{
            server 192.168.111.6;
            server  192.168.111.4;
            server  192.168.111.5;
}
```
3. proxy_pass http://aaa.com/aaa/  这里必须要加这个目录，不然就访问到根目录了。
4. 实际上，上述配置文件中， localtion /bbb/ 部分是可以省略掉的，因为后边的 location /  已经包含了/bbb/，所以即使我们不去定义  localtion /bbb/ 也是会访问到 bbb.com 的。 