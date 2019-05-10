#### mysql常用操作
```
查看都有哪些库 show databases；
查看某个库的表 use db；show tables；
查看表的字段 desc tb；
查看建表语句 show create table tb；
当前是哪个用户 select user();
当前库 select database()；
创建库 create database db1；
创建表 create table t1 (\`id\`int(4),\`name\` char(40));
查看数据库版本 select version();
查看mysql状态 show status;
修改mysql参数 show variables like 'max_connect%';set global max_connect_errors=1000;
查看mysql队列 show processlist；
创建普通用户并授权 grant all on \*.\* to user1 identified by '123456'
grant all on db1.* to 'user2'@'10.0.2.100' identified by '111222';
grant all on db1.* to 'user3'@'%' identified by '231222';
更改密码 UPDATE mysql.user SET password=PASSWORD("newpwd")  WHERE user='username';
查询 select count(*) from mysql.user;select * from mysql.db;select * from mysql.db where host like '10.0.%';
插入update db1.t1 set name='aaa' where id=1;
清空表 truncate table db1.t1;
删除表 drop table db1.t1;
删除数据库 drop datebase db1;
修复表 repair table tb1 [use frm]；
```
#### mysql备份与恢复
```
备份 mysqldump -uroot -p db >1.sql
恢复 musql -uroot -p db <1.sql
只备份一表 mysqldump -uroot -p db tb1 > 2.sql
备份时指定字符集 mysqldump -uroot -p --default-character-set=utf8 db > 1.sql
恢复时指定字符集 mysqldump -uroot -p --default-character-set=utf8 db < 1.sql
```
