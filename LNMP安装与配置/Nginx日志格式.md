#### Nginx日志格式

1. ​

```shell
og_format main '$remote_addr - $remote_user [$time_local] $request '
                    '"$status" $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';
```

2. ​

```shell
log_format main1 '$proxy_add_x_forwarded_for - $remote_user [$time_local] '
                      '"$request" $status $body_bytes_sent '
                      '"$http_referer" "$http_user_agent"';  //此日志格式为，ip不仅记录代理的ip还记录远程客户端真实IP。
```

3. ​

```shell
og_format  combined_realip  '$remote_addr $http_x_forwarded_for [$time_local] '
                 '$host "$request_uri" $status '
                 '"$http_referer" "$http_user_agent"';
```
也是记录代理ip以及源真实ip的一种日志格式 