####HA高可用集群####
HA即（high available）高可用，又被叫做双击热备，用于关键性业务。简单理解就是，有两台机器A和B，正常是A提供服务，B待命闲置，当A宕机或服务宕掉，会切换至B机器作为HA对应的服务。

下面使用heartbeat来做HA集群，并且把nginx服务作为HA对应的服务

试验准备：
两个机器，都是centos，网卡eth0 ，ip如下：
centos1 192.168.22.10
centos2 192.168.22.11

1. hostname设置好，分别为centos1和centos2
2. 关闭防火墙 iptables -F
关闭selinux：setenforce 0
3. vi /etc/hosts #增加内容如下
centos1 192.168.22.10
centos2 192.168.22.11

4. 安装epel扩展源
`yum -y install epel-release`
5. 两个机器都安装heartbeat/libnet
yum install -y heartbeat* libnet nginx
注：此处做实验用了nginx所以装nginx包

6. 主上（centos1）配置
cd /usr/share/doc/heartbeat-3.0.4
cp authkeys ha.cf haresources /etc/ha.d/
cd /etc/ha.d
vi authkeys #加入
```shell
auth 3
3 md5 Hello!
```
注：修改authkeys的权限，必须为600或400否则不能启动
chmod 600 authkeys
vi haresources #定义资源 加入
centos1 192.168.22.2/24/eth0:0 nginx
vi ha.cf #配置核心配置文件
```shell
debugfile /var/log/ha-debug #错误日志
logfile /var/log/ha-log #日志
logfacility local0
keepalive 2 #发送心跳信息的间隔
deadtime 30 #集群故障之后节点退出集群的时间间隔
warntime 10 #警告时间，检查到10秒无法连接就记录日志
initdead 60 #集群服务启动时的死亡时间间隔
udpport 694 #端口
ucast eth0 192.168.22.11 #通过单播直连对方机器 mcast多播
auto_failback on #自动切回
node centos1 #添加集群节点只能使用主机名
node centos2
ping 192.168.25.1 #仲裁地址
respawn hacluster /usr/lib/heartbeat/ipfail
```
注：如系统为64位，则应该为` respawn hacluster /usr/lib/heartbeat/ipfail `
7. 把主上（centos1）上的三个配置文件拷贝到从上（centos2）
cd /etc/ha.d
scp authkeys ha.cf haresources centos2:/etc/ha.d/

8. 在从上（centos2）编辑文件ha.cf
vi /etc/ha.d/ha.cf
更改ucast为centos1的地址
ucast eth0 192.168.22.10
9. 启动heartbeat
先主（centos1），后从（centos2）
`service heartbeat start`
10. 检查测试
ifconfig 看是否有eth0:0
ps aux|grep nginx 查看nginx进程是否启动
11. 测试1
主上故意禁ping
`iptables -I INPUT -p icmp -j DROP`
查看是否切到从上
12. 测试2
主上停止heartbeat服务
service heartbeat stop
查看是否切到从上
####heartbeat常见错误排查解决
1. 错误一：

[root@snale2 ha.d 11:55:37]#service heartbeat start

Starting High-Availability services: INFO: Resource is stopped
Heartbeat failure [rc=6]. Failed.
heartbeat: udpport setting must precede media statementsheartbeat[2299]: 2016/10/18_11:55:43 ERROR: Client child command [/usr/lib/heartbeat/ipfail] is not executable
heartbeat[2299]: 2016/10/18_11:55:43 ERROR: Heartbeat not started: configuration error.
heartbeat[2299]: 2016/10/18_11:55:43 ERROR: Configuration error, heartbeat not started.

解决方法
vim /etc/ha.d/ha.cf
将 respawn hacluster /usr/lib/heartbeat/ipfail 改成 respawn hacluster /usr/lib/heartbeat/ipfail //系统是64位的，所以路径应该在/usr/lib64/目录

2. 错误二：

[root@snale2 ha.d 12:01:03]#service heartbeat start
Starting High-Availability services: INFO: Resource is stopped
Heartbeat failure [rc=6]. Failed.

heartbeat: udpport setting must precede media statementsheartbeat[2464]: 2016/10/18_12:01:59 ERROR: Missing auth directive in keyfile [/etc/ha.d//authkeys]
heartbeat[2464]: 2016/10/18_12:01:59 ERROR: Authentication configuration error.

heartbeat[2464]: 2016/10/18_12:01:59 ERROR: Configuration error, heartbeat not started.

解决方法
vim /etc/ha.d/authkeys
改3 md5 Hello!为
auth 3
3 md5 Hello!

3. 错误三：

[root@snale2 ha.d 13:44:32]#service heartbeat start
Starting High-Availability services: INFO: Resource is stopped
Heartbeat failure [rc=6]. Failed.
heartbeat: udpport setting must precede media statementsheartbeat: baudrate setting must precede media statementsheartbeat[2641]: 2016/10/18_13:44:36 info: Pacemaker support: false

heartbeat[2641]: 2016/10/18_13:44:36 ERROR: Current node [snale2.localdomain] not in configuration!

heartbeat[2641]: 2016/10/18_13:44:36 info: By default, cluster nodes are named by `uname -n` and must be declared with a 'node' directive in the ha.cf file.

heartbeat[2641]: 2016/10/18_13:44:36 info: See also: http://Linux-ha.org/wiki/Ha.cf#node_directive

heartbeat[2641]: 2016/10/18_13:44:36 WARN: Logging daemon is disabled --enabling logging daemon is recommended

heartbeat[2641]: 2016/10/18_13:44:36 ERROR: Configuration error, heartbeat not started.

错误原因：在ha.cf配置中 node snale2 与 uname -n 得出的结果snale2.localdomain不一致，改成一直即可。
