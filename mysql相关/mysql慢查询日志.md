MySQL慢查询配置

1. 慢查询有什么用?

它能记录下所有执行超过long_query_time时间的SQL语句, 帮你找到执行慢的SQL, 方便我们对这些SQL进行优化.

2. 如何开启慢查询?

首先我们先查看mysql服务器的慢查询状态是否开启.执行如下命令:

show variables like '%quer%';

我们可以看到当前log_slow_queries状态为OFF, 说明当前并没有开启慢查询.

开启慢查询非常简单, 操作如下:

直接去my.conf中查看。

my.conf中的配置(放在[mysqld]下的下方加入)

[mysqld]

log-slow-queries = /usr/local/mysql/var/slowquery.log
long_query_time = 1  #单位是秒
log-queries-not-using-indexes

log-slow-queries: 代表MYSQL慢查询的日志存储目录, 此目录文件一定要有写权限；

Windows下需要写绝对路径，如：log-slow-queries="C:/Program Files/MySQL/MySQL Server 5.5/log/mysql-slow.log"

long_query_time: 最长执行时间. (如图, MSYQL将记录下所有执行时间超过2条的SQL语句, 此处为测试时间, 时间不应太小最好在5-10秒之内, 当然可以根据自己的标准而定);

log-queries-not-using-indexes    ：没有使用到索引的查询也将被记录在日志中
配置好以后重新启动一个MYSQL服务

使用sql语句来修改：不能按照my.conf中的项来修改的。修改通过"show VARIABLES like "%slow%" "
语句列出来的变量，运行如下sql：

set global log_slow_queries = ON;
set global slow_query_log = ON;
set global long_query_time=0.1; #设置大于0.1s的sql语句记录下来




慢查询日志文件的信息格式：
```
# Time: 130905 14:15:59         时间是2013年9月5日 14:15:59(前面部分容易看错哦,乍看以为是时间戳)
# User@Host: root[root] @  [183.239.28.174]  请求mysql服务器的客户端ip
# Query_time: 0.735883  Lock_time: 0.000078 Rows_sent: 262  Rows_examined: 262 这里表示执行用时多少秒，0.735883秒，1秒等于1000毫秒

SET timestamp=1378361759;  这目前我还不知道干嘛用的
show tables from `test_db`; 这个就是关键信息，指明了当时执行的是这条语句
```



设置毫秒级别与mysql版本的关系


很多网上资料显示，5.21之前的版本，在my.conf中的long_query_time最小只能设置为1(也就是最小1秒)。我自己歪打误撞，发现我通过其他方式可以实现。


"long_query_time = 0.1"这种方式我没试，因为数据库服务器在跑。需要重启。所以没试。我是通过全局变量设置实现慢日志查询记录的。

可以通过全局变量设置方式实现毫秒级别记录：

set global long_query_time = 0.01

我服务器上mysql版本为：5.1.60-log

我试过这种方式有效。

怎么测试自己的查询是否会被记录下来呢？
运行语句
select sleep(0.13);


我故意设置0.13秒延迟，然后这条语句按照预期(因为之前设置超过0.1秒)会被记录到日志文件中去。
