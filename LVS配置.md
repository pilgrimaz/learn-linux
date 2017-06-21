#### LVS 8种调度算法

最好参考此文章：[http://www.linuxvirtualserver.org/zh/lvs4.html](http://www.linuxvirtualserver.org/zh/lvs4.html)

Lvs的调度算法决定了如何在集群节点之间分布工作负荷。当director调度器收到来自客户端访问VIP的上的集群服务的入站请求时，director调度器必须决定哪个集群节点应该处理请求。Director调度器用的调度方法基本分为两类：

固定调度算法：rr，wrr，dh，sh

动态调度算法：wlc，lc，lblc，lblcr

| 算法 | 说明 |
| --- | --- |
| rr | 轮询算法，它将请求依次分配给不同的rs节点，也就是RS节点中均摊分配。这种算法简单，但只适合于RS节点处理性能差不多的情况 |
| wrr | 加权轮训调度，它将依据不同RS的权值分配任务。权值较高的RS将优先获得任务，并且分配到的连接数将比权值低的RS更多。相同权值的RS得到相同数目的连接数。 |
| Wlc | 加权最小连接数调度，假设各台RS的全职依次为Wi，当前tcp连接数依次为Ti，依次去Ti/Wi为最小的RS作为下一个分配的RS |
| Dh | 目的地址哈希调度（destination hashing）以目的地址为关键字查找一个静态hash表来获得需要的RS |
| SH | 源地址哈希调度（source hashing）以源地址为关键字查找一个静态hash表来获得需要的RS |
| Lc | 最小连接数调度（least-connection）,IPVS表存储了所有活动的连接。LB会比较将连接请求发送到当前连接最少的RS. |
| Lblc | 基于地址的最小连接数调度（locality-based least-connection）：将来自同一个目的地址的请求分配给同一台RS，此时这台服务器是尚未满负荷的。否则就将这个请求分配给连接数最小的RS，并以它作为下一次分配的首先考虑。 |

LVS调度算法的生产环境选型：

1、一般的网络服务，如http，mail，mysql等常用的LVS调度算法为：

​    a.基本轮询调度rr

​    b.加权最小连接调度wlc

​    c.加权轮询调度wrc

2、基于局部性的最小连接lblc和带复制的给予局部性最小连接lblcr主要适用于web cache和DB cache

3、源地址散列调度SH和目标地址散列调度DH可以结合使用在防火墙集群中，可以保证整个系统的出入口唯一。

实际适用中这些算法的适用范围很多，工作中最好参考内核中的连接调度算法的实现原理，然后根据具体的业务需求合理的选型。

#### LVS/NAT配置

三台服务器一台作为director，两台作为real server

director有一个外网IP\(192.168.0.11\)和一个内网IP（192.168.25.131），两个realserver上只有内网IP（192.168.25.132）和（192.168.25.133）并且需要把两个real server的内网网关设置为director的内网ip（192.168.25.131）

两个realserver上都安装httpd：yum install -y httpd （测试情况，实际两个服务器可做共享存储或者配置一样的数据及服务）

director 上安装ipvsadm：yum install -y httpd

director 上 vim /usr/local/sbin/lvs\_nat.sh  //增加

```shell
#!/bin/bash
#director服务器上开启路由转发功能：
echo 1 > /proc/sys/net/ipv4/ip_forward
#关闭icmp的重定向
echo 0 > /proc/sys/net/ipv4/conf/all/send_redirects
echo 0 > /proc/sys/net/ipv4/conf/default/send_redirects
echo 0 > /proc/sys/net/ipv4/conf/eth0/send_redirects
echo 0 > /proc/sys/net/ipv4/conf/eth1/send_redirects

#director 设置nat防火墙
iptables -t nat -F
iptables -t nat -X
iptables -t nat -A POSTROUTING -s 192.168.25.0/24 -j MASQUERADE
#director设置ipvsadm
IPVSADM='/sbin/ipvsadm'
$IPVSADM -C
$IPVSADM -A -t 192.168.0.11:80 -s lc -p 300
$IPVSADM -a -t 192.168.0.11:80 -r 192.168.25.132:80 -m -w 1
$ipvsadm -a -t 192.168.0.11:80 -r 192.168.25.133:80 -m -w 1
```

直接运行这个脚本就可以完成lvs/nat的配置了：

/bin/bash/usr/local/sbin/lvs\_nat.sh

通过浏览器测试两台机器上的httpd内容

#### LVS/DR配置

三台机器：

director\(eth0192.168.0.11,vip eth0:0: 192.168.0.100）

real server1\(eth0 rip:192.168.0.21,vip lo:192.168.0.100\)

real server2\(eth0 rip:192.168.0.22,vip lo:192.168.0.100\)

Director 上vim /usr/local/sbin/lvs\_dr.sh  //增加

```shell
#!/bin/bash
echo 1 > /proc/sys/net/ipv4/ip_forward
ipv=/sbin/ipvsadm
vip=192.168.0.100
rs1=192.168.0.21
rs2=192.168.0.22
ifconfig eth0:0 $vip broadcast $vip netmask 255.255.255.255 up
route add -host $vip dev eth0:0
$ipv -C
$ipv -A -t $vip:80 -s rr
$ipv -a -t $vip:80 -r $rs1:80 -g -w 1
$ipv -a -t $vip:80 -r $rs2:80 -g -w 1
```

两台rs上：vim /usr/local/sbin/lvs\_dr\_rs.sh

```shell
#!/bin/bash
vip=192.168.0.100
ifconfig lo:0 $vip broadcast $vip netmask 255.255.255.255 up
route add -host $vip lo:0
echo "1" >/proc/sys/net/ipv4/conf/lo/arp_ignore
echo "2" >/proc/sys/net/ipv4/conf/lo/arp_announce
echo "1" >/proc/sys/net/ipv4/conf/all/arp_ignore
echo "2" >/proc/sys/net/ipv4/conf/all/arp_announce
```

然后director上执行：bash /usr/local/sbin/lvs\_dr.sh

两台rs上执行： bash /usr/local/sbin/lvs\_dr\_rs.sh

windows下浏览器测试访问

#### LVS/DR + keepalive配置

注意：前面虽然我们已经配置过一些操作，但是下面我们使用keepalive操作和之前的操作是有些冲突的，所以若是之前配置过DR，请首先做如下操作：

dr上执行：

$ipv -C

ifconfig eth0:0 down

前面的lvs虽然已经配置成功也实现了负载均衡，但是我们测试的时候发现，当某台real server把httpd进程停掉，那么director照样会把请求转发过去，这样就造成了某些请求不正常。所以需要有一种机制用来检测real server的状态，这就是keepalived。它的作用除了可以检测rs状态外，还可以检测备用director的状态，也就是说keepalived可以实现ha集群的功能，当然了也需要一台备用director。

备用director也需要安装一下keepalived软件

yum install -y keepalived

安装好后，编辑配置文件

vim /etc/keepalived/keepalived.conf  //加入如下：

```shell
vrrp_instance VI_1 {
    state MASTER   #备用服务器上为BACKUP
    interface eth0
    virtual_router_id 51
    priority 100    #备用服务器上为90
    advert_int 1
    authentication{
      auth_type PASS
      auth_pass 1111
      }
      virtual_ipaddress {
        192.168.0.100
      }
    }
    virtual_server 192.168.0.100 80 {
      delay_loop 6         #(每隔10秒查询realserver状态)
      lb_algo wlc           #(lvs算法)
      lb_kind DR           #(Direct Route)
      persistence_timeout 60   #(同一IP的连接60秒内被分配到同一台realserver)
      protocol TCP         #(用TCP协议检查realserver状态)

      real_server 192.168.0.21 80 {
        weight 100           #（权重）
        TCP_CHECK {
          connect_timeout 10   #(10秒无响应超时)
          nb_get_retry 3
          delay_before_retry 3
          connect_port 80
          }
        }
      real_server 192.168.0.22 80 {
        weight 100
        TCP_CHECK {
          connect_timeout 10
          nb_get_retry 3
          delay_before_retry 3
          connect_port 80
        }
      }
    }
```
注： persistence_timeout  # 会话保持时间，单位是秒，这个选项对于动态网页是非常有用的，为集群系统中session共享提供了一个很好的解决方案。有了这个会话保持功能，用户的请求会被一直分发到某个服务节点，直到超过这个会话保持时间。需要注意的是，这个会话保持时间，是最大无响应超时时间，也就是说用户在操作动态页面时，如果在50秒内没有执行任何操作，那么接下来的操作会被分发到另外节点，但是如果一直在操作动态页面，则不受50秒的时间限制，**如果轮巡则该值必须为0**

以上为主director的配置文件，从director的配置文件只需要修改

state MASTER -&gt; state BACKUP

priority 100 -&gt;proiority 90

配置完keepalived后，需要开启端口转发（主从都要做）：

echo 1 &gt; /proc/sys/net/ipv4/ip\_forward

然后，两个rs上执行 /usr/local/sbin/lvs\_dr\_rs.sh 脚本

最后，两个director上启动keepalived服务（先主后从）：

/etc/init.d/keepalived start

另外，需要注意的是，启动keepalived服务会自动生成vip和ipvsadm规则，不需要再去执行上面提到的 /usr/local/sbin/lvs\_dr.sh脚本。

