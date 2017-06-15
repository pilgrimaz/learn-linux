#### Nginx日志按日切割脚本
```shell
#! /bin/bash
datedir=`date +%Y%m%d`
/bin/mkdir  /home/logs/$datedir >/dev/null 2>&1
/bin/mv /home/logs/*.log /home/logs/$datedir
/bin/kill -HUP `cat /var/run/nginx.pid`
```