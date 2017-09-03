###mariadb10.2.8配置主从  

>  操作系统:CentOS Linux release 7.3.1611 (Core)
>  数据库:MariaDB-10.2.8-linux-glibc_214-x86_64
>
>  主从一机，分别用了3306和3307监听端口，my.cnf放置于datadir目录下

**1.单向主从同步**
单机装双数据库或者单机启动双实例多配置文件可参考网上文档


**2.在MariaDB(MySQL)配置文件中修改或添加以下信息**

vim /etc/my.cnf

主从通用配置

>  ```
>  binlog-format = row #二进制日志记录的模式（高版本默认开启）
>  binlog-checksum = CRC32 #可使主机为写入二进制日志的事件写入校验（高版本默认开启）
>  sync-master-info = 1 #MariaDB依靠操作系统将master.info文件刷新到磁盘。
>  sync_relay_log_info = 1 #MariaDB依靠操作系统将relay-log.info文件刷新到磁盘。
>  expire_logs_days = 7 #日志文件过期天数,默认是 0,表示不过期 
>  master-verify-checksum = 1 #主服务器效验
>  slave-sql-verify-checksum = 1 #从服务器效验
>  ```

**3.主服务器Master除了通用配置外,还需要加入以下代码**

>  ```
>  server-id = 56 #MySQL服务器ID,不重复
>  log-bin = mysql-bin #二进制日志（默认开启）
>  sync-binlog = 1 #主服务器进行设置,用于事务安全
>  log-bin-index = mysql-bin
>  ```

**4.从服务器Slave除了通用配置外,还需要加入以下代码**

>  ```
>  server-id = 163
>  relay-log = relay-bin #中继日志
>  slave-parallel-threads = 2 #设定从服务器的SQL线程数
>  #replicate-do-db = renwoleblogdb#复制指定的数据库,多个写多行
>  replicate-ignore-db = mysql #不备份的数据库,多个写多行
>  relay_log_recovery = 1 #从站崩溃后可以使用,防止损坏的中继日志处理。
>  log-slave-updates = 1 #slave将复制事件写进自己的二进制日志
>  relay-log-index = relay-bin
>  ```

此外Mysql从服务器没有必要开启二进制日志，但是在一些情况下,必须设置,例如;如果slave为其它slave的master，必须设置bin_log。我这里就默认开启。

**5.以上只是简单的介绍了每个参数的作用,这些参数的具体设置还需根据用户的实际情况进行相关调整,具体可到官方了解**

>  《复制和二进制日志服务器系统变量》
>  https://mariadb.com/kb/en/mariadb/replication-and-binary-log-server-system-variables/

关于系统变量的兼容性，可参阅官方

>  《MariaDB与MySQL兼容性》
>  https://mariadb.com/kb/en/mariadb/mariadb-vs-mysql-compatibility/

**6.主服务器Master授权配置**

主MariaDB服务器上创建专用账号并授权数据库权限,以及从服务器IP的远程访问

>  ```
>  # mysql -uroot -p
>  Enter password:【输入你的MySQL密码回车】
>  MariaDB [(none)]> GRANT REPLICATION SLAVE ON *.* TO 'rep'@'%' IDENTIFIED BY 'rep123456'; //创建Slave专用备份账号
>  MariaDB [(none)]> flush privileges; //刷新MySQL权限
>  MariaDB [(none)]> SELECT DISTINCT CONCAT('User: ''',user,'''@''',host,''';') AS query FROM mysql.user; //查看授权情况
>  MariaDB [(none)]> flush tables with read lock; //锁定数据库防止master值变化
>  MariaDB [(none)]> show master status; //获取master状态值
>  +-----------------+----------+------------+-----------------+
>  | File |Position |Binlog_Do_DB|Binlog_Ignore_DB |
>  +-----------------+----------+------------+-----------------+
>  | mysql-bin.000006| 627 | | |
>  +-----------------+----------+------------+-----------------+
>  1 row in set (0.00 sec)
>  ```

**7.一旦获取了备份时正确的Binlog位点（文件名和偏移量），那么就可以用BINLOG_GTID_POS()函数来计算GTID**

>  ```
>  MariaDB [(none)]> SELECT BINLOG_GTID_POS("mysql-bin.000006", 627);
>  +------------------------------------------+
>  | BINLOG_GTID_POS('mysql-bin.000006', 627) |
>  +------------------------------------------+
>  | 0-56-4 |
>  +------------------------------------------+
>  1 row in set (0.01 sec)
>  ```

**8.从服务器Slave配置**

正如官方所说从MariaDB 10.0.13版本开始，mysqldump会自动完成这个工作，并且把GTID的写在导出文件中，只要设置 –master-data 或 –dump-slave 的同时设置 –gtid 即可。

这样的话新的SLAVE就可以通过设置 @@gtid_slave_pos 的值来设定复制的起始位置，用 CHANGE MASTER 把这个值传给主库，然后开始复制：

>  ```
>  # mysql -uroot -p
>  Enter password:【输入你的MySQL密码】
>  MariaDB [(none)]> SET GLOBAL gtid_slave_pos = "0-56-4";
>  MariaDB [(none)]> change master to master_host='127.0.0.1',MASTER_PORT =3306,master_user='rep',master_password='rep123456',master_use_gtid=slave_pos; //进行主从授权
>  MariaDB [(none)]> START SLAVE; //启动Slave
>  MariaDB [(none)]> show slave status\G
>  *************************** 1. row ***************************
>   Slave_IO_State: Waiting for master to send event
>   Master_Host: 10.10.10.56
>   Master_User: renwoleuseracc
>   Master_Port: 3306
>   Connect_Retry: 60
>   Master_Log_File: mysql-bin.000006
>   Read_Master_Log_Pos: 627
>   Relay_Log_File: relay.000035
>   Relay_Log_Pos: 537
>   Relay_Master_Log_File: mysql-bin.000006
>   Slave_IO_Running: Yes
>   Slave_SQL_Running: Yes
>   ...
>   ...
>   ...
>   Using_Gtid: Slave_pos
>   Gtid_IO_Pos: 0-56-4
>  ```

**9.如果 Slave_IO_Running 与 Slave_SQL_Running 都为YES,表明从服务已经运行，Using_Gtid列判断GTID值是否一致。**
说明：

>  master_host 表示master授权地址
>  MASTER_PORT MySQL端口
>  master_user 表示master授权账号
>  master_password 表示密码
>  master_use_gtid GTID变量值

**10.接下来解锁主服务器数据库表**

>  ```
>  MariaDB [(none)]> unlock tables; //解锁数据表
>  MariaDB [(none)]> show slave hosts; //查看从服务器连接状态
>  MariaDB [(none)]> show global status like "rpl%"; //查看客户端
>  ```

**11.从服务器Slave查看relay的所有相关参数**

>  ```
>  MariaDB [(none)]> show variables like '%relay%';
>  ```

**12.主从已经配置完成。现在无论在主服务器上增、改、删、查，都会同步到从服务器，根据自己的需求进行相关测试即可。**
关于master slave重置语法

重置master的核心语法

RESET MASTER; 意思是执行 RESET MASTER 将删除所有二进制日志文件，并创建一个空白的二进制日志文件，数字后缀为.000001，RESET MASTER 不会影响SLAVE服务器上的工作状态，所以执行这个命令会导致Slave找不到Master的binlog，从而造成同步失败。

重置slave的核心语法

RESET SLAVE; 含义是；RESET SLAVE 将清除slave上的同步位置并删除所有旧的同步中继日志文件，但重置前必须先停止slave服务（STOP SLAVE）

以后有时间，我将介绍基于GTID的半同步主从的文章，这也是在生产中所需要记录下来，以备以后使用。