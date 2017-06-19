```shell
debugfile /var/log/ha-debug   #错误日志
logfile /var/log/ha-log    #日志
logfacility  local0    
keepalive 2    #检测时间 2秒检测一次
deadtime 30    #超过30秒检查不到表示挂了
warntime 10    #检查到10秒无法连接就记录日志
initdead 60    #预留时间
udpport 694    #端口
ucast eth0 192.168.25.130    #对方机器
auto_failback on    #自动切回
node CentOS128      
node centOS130
ping 192.168.25.1     #仲裁地址
respawn hacluster /usr/lib/heartbeat/ipfail
```