#### MYSQL相关配置

[mysqld]

socket = /tmp/mysql.sock    #为mysql客户程序与服务器之间本地通信指定一个套接字文件（linux下默认是在/var/lib/mysql/mysql.sock文件）  
port = 3306    #指定mysql监听的端口  
key_buffer = 384M    #key_buffer是用于索引块的缓冲区大小，增加它可得到更好处理的索引(对所有读和多重写)。key_buffer的大小视内存大小而定。   
table_cache = 512   #为所有线程打开表的数量。增加该值能增加mysqld要求的文件描述符的位置。可以避免频繁的打开数据表产生的开销  
sort_buffer_size = 2M    #每个需要进行排序的线程分配该大小的一个缓冲区。增加这值加速ORDER BY或GROUP BY操作      
     注意：该操作对应的分配内存是每连接独占！如果有100个连接，那么实际分配的总共排序缓冲区大小为100*2=200MB   
read_buffer_size = 2M    #读查询操作所能使用的缓冲区大小。和sort_buff_size一样，该参数对应的分配内存也是每连接独享。   
query_cache_size = 32M   #指定mysql查询结果缓冲区的大小   
read_rnd_buffer_size = 8M  #改参数在使用行指针排序之后，随机读用的。   
myisam_sort_buffer_size = 64M #MyISAM表发生变化时重新排序所需的缓冲   
thread_concurrency = 8  #最大并发线程数，取值为服务器逻辑CPU数量\*2，如果CPU支持H.T超线程，再\*2   
thread_cache = 8  #缓存可重用的线程数    
skip-locking    #避免mysql的外部锁定，减少出错几率增强稳定性。   
wait_timeout = 8   #表示空闲的连接超时时间，默认28800s，这个参数适合interactive_timeout一起使用的，也就是说要想让wait_timeout生效，必须同时设置interactive_time   
interactive_timeout = 8   
long_query_time = 1  #慢查询日志的超时时间    
log_slow_queries = /path/to/slow_queries  #慢查询日志路径，必须配合上面的参数一同使用    


####mysql调优   
MySQL调优可以从几个方面来做：   
1. 架构层：    
做从库，实现读写分离；   

2.系统层次：   
增加内存；   
给磁盘做raid0或者raid5以增加磁盘的读写速度；    
可以重新挂载磁盘，并加上noatime参数，这样可以减少磁盘的i/o;     

3. MySQL本身调优：    
(1) 如果未配置主从同步，可以把bin-log功能关闭，减少磁盘i/o    
(2) 在my.cnf中加上skip-name-resolve,这样可以避免由于解析主机名延迟造成mysql执行慢   
(3) 调整几个关键的buffer和cache。调整的依据，主要根据数据库的状态来调试。如何调优可以参考5.   

4. 应用层次：   
查看慢查询日志，根据慢查询日志优化程序中的SQL语句，比如增加索引    

5. 调整几个关键的buffer和cache    
1) key_buffer_size  首先可以根据系统的内存大小设定它，大概的一个参考值：1G以下内存设定128M；2G/256M; 4G/384M;8G/1024M；16G/2048M.这个值可以通过检查状态值Key_read_requests和 Key_reads,可以知道key_buffer_size设置是否合理。比例key_reads / key_read_requests应该尽可能的低，至少是1:100，1:1000更好(上述状态值可以使用SHOW STATUS LIKE ‘key_read%’获得)。注意：该参数值设置的过大反而会是服务器整体效率降低!    

2) table_open_cache 打开一个表的时候，会临时把表里面的数据放到这部分内存中，一般设置成1024就够了，它的大小我们可以通过这样的方法来衡量： 如果你发现 open_tables等于table_cache，并且opened_tables在不断增长，那么你就需要增加table_cache的值了(上述状态值可以使用SHOW STATUS LIKE ‘Open%tables’获得)。注意，不能盲目地把table_cache设置成很大的值。如果设置得太高，可能会造成文件描述符不足，从而造成性能不稳定或者连接失败。    

3) sort_buffer_size 查询排序时所能使用的缓冲区大小,该参数对应的分配内存是每连接独占!如果有100个连接，那么实际分配的总共排序缓冲区大小为100 × 4 = 400MB。所以，对于内存在4GB左右的服务器推荐设置为4-8M。    

4) read_buffer_size 读查询操作所能使用的缓冲区大小。和sort_buffer_size一样，该参数对应的分配内存也是每连接独享!    

5) join_buffer_size 联合查询操作所能使用的缓冲区大小，和sort_buffer_size一样，该参数对应的分配内存也是每连接独享!    

6) myisam_sort_buffer_size 这个缓冲区主要用于修复表过程中排序索引使用的内存或者是建立索引时排序索引用到的内存大小，一般4G内存给64M即可。    

7) query_cache_size MySQL查询操作缓冲区的大小，通过以下做法调整：SHOW STATUS LIKE ‘Qcache%’; 如果Qcache_lowmem_prunes该参数记录有多少条查询因为内存不足而被移除出查询缓存。通过这个值，用户可以适当的调整缓存大小。如果该值非常大，则表明经常出现缓冲不够的情况，需要增加缓存大小;Qcache_free_memory:查询缓存的内存大小，通过这个参数可以很清晰的知道当前系统的查询内存是否够用，是多了，还是不够用，我们可以根据实际情况做出调整。一般情况下4G内存设置64M足够了。    

8) thread_cache_size 表示可以重新利用保存在缓存中线程的数，参考如下值：1G  —> 8 2G  —> 16 3G  —> 32  >3G  —> 64    
除此之外，还有几个比较关键的参数：    

9) thread_concurrency 这个值设置为cpu核数的2倍即可    

10) wait_timeout 表示空闲的连接超时时间，默认是28800s，这个参数是和interactive_timeout一起使用的，也就是说要想让wait_timeout 生效，必须同时设置interactive_timeout，建议他们两个都设置为10     

11) max_connect_errors 是一个MySQL中与安全有关的计数器值，它负责阻止过多尝试失败的客户端以防止暴力破解密码的情况。与性能并无太大关系。为了避免一些错误我们一般都设置比较大，比如说10000    

12) max_connections 最大的连接数，根据业务请求量适当调整，设置500足够     

13) max_user_connections 是指同一个账号能够同时连接到mysql服务的最大连接数。设置为0表示不限制。通常我们设置为100足够      
