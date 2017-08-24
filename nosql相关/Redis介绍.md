[TOC]



####Redis介绍
- redis是一个key-value存储系统，官方站点http://redis.io
- 和memcached类似，但支持数据数据持久化。
- 支持更多value类型，除了 string外，还支持hash、lists(链表)、sets(集合)和sorted sets(有序集合)几种数据类型
- redis使用了两种文件格式：全量数据(RDB)和增量请求(AOF)。全量数据格式是把内存中的数据写入磁盘，便于下次读取文件进行加载。增量请求文件则是把内存中的数据序列化为操作请求，用于读取文件进行replay得到数据
- redis的存储分为内存存储、磁盘存储和log文件三部分。


####redis持久化
- redis提供了两种持久化的方式，分别是RDB(Redis database)和AOF(Append Only File)。
- RDB，就是在不同的时间点，将redis存储的数据生成快照并存储到磁盘等介质上
- AOF，则是换了一个角度来实现持久化，那就是将redis执行过的所有写指令记录下来，在下次redis重新启动时，只要把这些指令从前到后在再重复执行一遍，就可以实现数据恢复了。
- 其实RDB和AOF两种方式也可以同时使用，在这种情况下，如果redis重启的话，则会优先采用AOF方式来进行数据恢复，这是因为AOF方式的数据恢复完整度更高。
- 如果你没有数据持久化的需求，也完全可以关闭RDB和AOF方式，这样的话，redis将变成一个纯内存数据库，就像memcache一样


####redis配置文件
config get *    //查看所有的配置项
config get timeout 


- **daemonize no** #默认情况下，redis并不是以daemon形式来运行的。通过daemonize配置项可以控制redis的运行形式。
- **pidfile/path/to/rendis.pid**  #当以daemon形式运行时，redis会生成一个pid文件，默认会生成在/var/run/redis.pid
- **bind 192.168.1.2**  10.8.4.2   #指定绑定的ip，可以有多个
- **port 6379**  #指定监听端口
- **unixsocketperm 775**  #当监听socket时可以指定权限为755
- **timeout 0**  #当一个redis-client一直没有请求发向server端，那么server端有权主动关闭这个连接，可以通过timeout来设置"空闲超时时限“，表示永不关闭。
- **tcp-keeplive 0**  #TCP连接保活策略，可以通过tcp-keepalive配置项来进行设置，单位为秒，加入设置为60秒，则server端会每60秒连接空闲的客户端发起一次ACK请求，以检查客户端是都已经挂掉，对于无响应的客户端则会关闭其连接。如果设置为0，则不会进行保活检测。
- **loglevel notice**  #日志级别，有四种debug，verbose，notice，warning
- **logfile** #定义日志路径
- **syslog-ident redis** #如果希望日志打印到syslog中，通过syslog-enable来控制。另外，syslog-ident还可以让你指定syslog里的日志标志
- **syslog-facility local0** #指定syslog的设备，可以使USER或者local0-local7
- **databases 16**  #设置数据库的总数量，select n选择数据库，0-15 


####redis快照配置(rdb持久化)
- **save 900 1** #表示每15分钟至少有一个key改变，就触发一次持久化
- **save 300 10** #表示每5分钟且至少有10个key改变，就触发一次持久化
- **save 60 10000** #表示每60秒至少有10000个key改变，就触发一次持久化
- **save ""**  #为空，这样可以禁用rdb持久化
- **stop-writes-on-bgsave-error yes** #rdb持久化写入磁盘避免不了会出现失败的情况，默认一旦出现失败，redis会马上停止写操作。如果你觉得无所谓，那就可以使用该选项关闭这个功能。
- **rdbcompression yes** #是否要压缩
- **rdbchecksum yes** #是否进行数据检验
- **dbfilename dump rdb** #定义快照文件的名字
- **dir ./**   #定义快照文件存储路径 
- **requirepass foobared** #设置redis-server的密码
- **rename-command CONFIG  zhangsan.config** #将CONFIG命令更名为zhangsan.config ,这样可以避免误操作，但如果使用了AOF持久化，建议不要启用该功能。
- **rename-command CONFIG ""**  #也可以后面定位为空，这样就禁掉了该CONFIG命令。
- **maxclients 10000**  #最大客户端连接数
- **maxmemory <bytes>**  #设定最大内存使用数，单位是byte
- **maxmemory-policy noeviction**  #指定内存移除规则
- **maxmemory-samples 5** #LRU算法和最小TTL算法都并非是精确的算法，而是估算值，所以你可以设置样本的大小。假如redis默认会检查三个key并选择其中LRU的哪个，那么你可以改变这个key样本的数量。

####redis (AOF持久化先关配置)
- **appendonly no**  如果是yes，则开启aof持久化
- **appendfilename "appendonly.aof**  #指定aof文件名字，保存在dir参数指定的目录
- **appendfsync everysec**  #指定fsync()调用模式，有三种no(不调用fsync)，always(每次写都会调用fsync)，everysec(每秒钟调用一次fsync)。第一种最快，第二种数据最安全，但性能会差一些，第三种为这种方案，默认为第三种。
- **no-appendfsync-on-rewrite no**  #使用no可避免当写入量非常大时的磁盘io阻塞
- **auto-aof-rewrite-percentage 100**  #规定什么情况下回触发aof重写。该值为一个比例，10表示当aof文件增幅达到10%是则会触发重写机制。
- **uto-aof-rewrite-min-size 64mb**  #重写会有一个条件，就是不能低于64mb


####redis 慢日志相关配置
- 针对慢日志，可以设置两个参数，一个执行时长，单位是微秒，另一个是慢日志的长度。当一个新的命令被写入日志时，最老的一条会从命令日志队列中被移除。
- **slowlog-log-slower-than 10000**  #慢于10000ms则记录日志
- **slowlog-max-len 128**  #日志长度


####redis主从配置